<template>
    <div :class="['request', method.toLowerCase(), isOpened]">

        <div :class="['request-item']" @click="open = !open">
            <span class="method">{{method}}</span>
            <span class="path">{{url}}</span>
            <span class="description">{{description}}</span>   
        </div>
        <params-table :params="sourceList" :resource="resource" v-show="open" />
    </div>
</template>

<script>

import ParamsTable from './params-table.vue'

export default {
    props: {
        description: {
            type: String,
            default: ''
        },
        url: {
            type: String,
            default: '/'            
        },
        params: { 
            type: Array,
            default: () => { 
                return []
            }
        }, 
        headers: { 
            type: Array,
            default: () => { 
                return [] 
            }
        },         
        path: { 
            type: Array,
            default: () => { 
                return [] 
            }
        },      
        body: {
            type: Object,
            default: () => { 
                return { } 
            }
        },
        method: {
            type: String,
            default: 'GET'
        },
        resource:{
            type: Object,
            default: () => { 
                return { } 
            }
        }

    },

    components: {
        ParamsTable
    },

    data () {
        return {

            open: false,
            spec: this.$parent.spec,
            scheme:this.$parent.scheme,
            dataHeaders: this.parseHeaders(this.headers),
            dataBody: this.parseBody(),
            dataPath: this.parsePath(this.path),
            dataQuery: this.parseQuery(this.params),
        }
    },
    computed: {

        isOpened () {
            return this.open
        },

        host () {
            return this.$parent.host
        },

        sourceList () {

            let sourceList = [].concat(this.dataPath, this.dataHeaders, this.dataBody, this.dataQuery)
          
            return sourceList
        }

    },
    methods: {


        parsePath () {


           const paths =  this.params.filter(param=>{
                return param.in == "path";
            });


            
            return paths.map(obj => {
                
                // console.log("******");
                

                let key = obj.name;

                let object = { key : key, value: obj};


                //console.log(object);


                return Object.assign({
                    source: 'path',
                    required: true,
                    type: 'string',
                    description: ''
                }, this.parseItems(obj))
            })
        },

        /**
         * request headers
         * 
         * {
         *  Authorization: 'Bearer {{access_token}}'
         * }
         * 
         * {{xxx}} 로 시작되는건 외부에서 입력받을 수 있는 변수로 대체됨 
         * 
         */
        parseHeaders (headers) {
            return headers.map(obj => {

                obj.params = this.parseHeaderValue(obj.description)

                return Object.assign({
                    source: 'header',
                    type: 'string',
                    required: true,
                    description: ''
                }, this.parseItems(obj))
            })
        },       
        /**
         * body 
         * 
         * {
         *  data : { a: 1, b: 2, c: 3 },
         *  contentType: 'application/json' 
         * }
         * 
         * or 
         * 
         * {
         *  data : "{ a: 1, b: 2, c: 3 }",
         *  contentType: 'text/plain' 
         * }
         * 
         */        
        parseBody () {



           let body =  this.params.filter(param=>{
                return param.in == "body";
            })[0];



            if (!body) return [];

            let properties = body.schema.properties;

            if (!properties) return [];


            if (properties.body && properties.body.type === 'string') {
                body.dataValue = properties.body.properties + "" 
            } else if (properties.body && properties.body.type === 'object') {
                body.dataValue = JSON.stringify(this.parseExample(properties.body.properties), null, 4)
            } 
            else{
                body.dataValue = JSON.stringify(this.parseExample(properties), null, 4)
            }



            return [
                Object.assign({
                    source: 'body',
                    key: 'body',
                    contentType: 'application/json',
                    required: false,
                    description: ''
                }, body)
            ]

        },

        parseExample(obj){

            let nObj = {};
            for (var key in obj) {
                if (obj.hasOwnProperty(key)) {

                    if(obj[key].type == "object"){
                        nObj[key] = this.parseExample(obj[key]);
                    }
                    if(obj[key].type == "array"){
                        let arr = [];
                        let items = obj[key].items;

                            if(items.type == "string"){
                                arr.push(items.example?items.example:items.type);
                            }
                            else if(items.type == "object"){
                                arr.push(this.parseExample(items.properties));
                            }

                        nObj[key] = arr;
                    }
                    else{
                        nObj[key] =  obj[key].example?obj[key].example:obj[key].type;
                    }
                    
                }
            }

            

            return nObj;


        },

        /**
         * query string 
         * 
         * [
         *   { key : 'xxx', value : 'xxx', type: 'string', defalut: 'xxx' , description : 'xxx', required: true },
         * ]
         * 
         */
        parseQuery (params) {


            const query =  this.params.filter(param=>{
                return param.in == "query";
            });


            return query.map(key => {
                
        
                return Object.assign({
                    source: 'query',
                    required: true,
                    type: 'string',
                    description: ''
                }, key)
            })


           
        },        
        parseHeaderValue (headerValue) {
            const arr = headerValue.match(/\{\{(.*)\}\}/g)
            return arr.map(it => {
                let key = it.replace(/\{/g, '').replace(/\}/g, '')
                return {key, value: ''}
            })
        },
        parseItems (item) {
            if (!item.items) return item 

            item.items = (item.items || []).map( (it, index) => {

                if (typeof it === 'string' ) {
                    it = { text: it, value: it }
                }

                if (index === 0) {
                    it.selected = true 
                    item.inputValue = it.value 
                }

                return it
            })

            return item
        }
    }
}
</script>

<style scoped lang="scss">
.request {
    border: 1px solid #000;
    border-radius: 4px;
    margin: 0 0 15px;

    .request-item {
        padding: 5px;
        cursor: pointer;
        display: flex;
        align-items: center;
    }

    // &.post {
    //     border-color: #49cc90;
    //     background: rgba(73,204,144,0.1);

    //     .method {
    //         background: #49cc90;
    //     }
    // }

    // &.put {
    //     border-color: #fca130;
    //     background: rgba(252,161,48,0.1);

    //     .method {
    //         background: #fca130;
    //     }
    // }

    // &.get {
    //     border-color: #61affe;
    //     background: rgba(97,175,254,0.1);

    //     .method {
    //         background: #61affe;
    //     }
    // }    

    // &.delete {
    //     border-color: #f93e3e;
    //     background: rgba(249,62,62,0.1);

    //     .method {
    //         background: #f93e3e;
    //     }
    // }


    &.post {
        border-color: #49cc90;
        background: rgba(42, 176, 98, 0.4);

        .method {
            background: #49cc90;
        }
    }

    &.put {
        border-color: #fca130;
        background: rgba(254, 131, 26, 0.4);

        .method {
            background: #fca130;
        }
    }

    &.get {
        border-color: #4687d8;
        background: rgba(70, 135, 216, 0.5);

        .method {
            background: #4687d8;
        }
    }    

    &.delete {
        border-color: #f93e3e;
        background: rgba(249, 62, 62, 0.4);

        .method {
            background: #f93e3e;
        }
    }    

    .method {
        font-size: 14px;
        font-weight: 700;
        min-width: 80px;
        padding: 6px 15px;
        text-align: center;
        border-radius: 3px;
        background: #000;
        text-shadow: 0 1px 0 rgba(0,0,0,0.1);
        font-family: sans-serif;
        color: #fff;
        text-transform: uppercase;
        box-sizing: border-box;
    }

    .path {
        font-size: 16px;
        display: flex;
        flex: 0 3 auto;
        align-items: center;
        word-break: break-all;
        padding: 0 10px;
        font-family: monospace;
        font-weight: 600;
        color: #3b4151;
        box-sizing: border-box;
    }

    .description {
        font-size: 13px;
        flex: 1;
        font-family: sans-serif;
        color: #3b4151;
    }
}
</style>
