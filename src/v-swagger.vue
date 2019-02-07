<template>
    <div class="api">
        <span v-if="error">{{ error }}</span>
        <section v-if="api">
        <div class="header" @click="open = !open">
            <span class="title">{{api.info.title}}</span>            
            <span class="host">{{api.host}}</span>            
            <!-- <span class="description">{{api.description}}</span> -->
        </div>

        <div class="table" v-show="isOpen">

            <section class="header">
                <div class="left">
                    <label for="scheme">Schemes</label>
                    <select class="select" v-model="scheme" id="scheme">
                        <option v-for="(scheme, index) in api.schemes" :key="index" :value="scheme">{{scheme}}</option>
                    </select>
                </div>
                <div class="right">
                         <label for="authorization">Authorization</label>
                        <select class="select" v-model="authorization" id="authorization">
                            <option value="">Choose</option>
                            <option v-for="(client, index) in authclients" :key="index" :value="client">{{client.clientName}}</option>
                        </select>
                </div>
            

            </section>


            <section v-bind:key="index" v-for="(tag, index) in tags">
                <h3>{{tag.name}}</h3>
                <p>{{tag.description}}</p>

                <div v-bind:key="path" v-for="(resources, path) in paths[tag.name]"> 
                    <!-- <h4>{{path}}</h4> -->
                    <slot></slot>
                    <request v-for="(item, method) in resources" 
                    
                        :key="method"
                        :method="method"
                        :url="path"
                        :description="item.summary"
                        :headers="item.headers"
                        :path="item.parameters"
                        :params="item.parameters"
                        :body="item.responses"
                        :resource="item"
                    />
                </div>
                
            </section>    
          
             
        </div>
        
        <slot></slot>

        <div class="table definitions">
            <section class="header">Models</section>

            <section class="models">
                <div class="model" 
                    v-bind:key="name" v-for="(model, name) in api.definitions" >
                   
                    <model :model="model" :name="name" />

                </div>
            </section>
            

        </div>

        </section>


    </div>
</template>

<script>

import request from './request.vue'
import model from './model.vue'


var SwaggerParser = require('swagger-parser');

export default {
     props: {
        spec: {
            type: Object,
            default: {}
        },
             
        authclients: { 
            type: Array,
            default: () => { 
                return [] 
            }
        }      
       

    },
    data () {

        return {
            scheme:'https',
            authorization:'',
            specInfo: this.spec,
            open: this.spec.opened || true,
            isCollapsed: false,
            api:'',
            error: undefined,
            tags:[{
                name:'default',
                description:''
            }],
            paths:[]
        }
    },
    methods:{
        toggleCollapsation(model) {
            return model.isCollapsed = !model.isCollapsed;
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


        }
    },
    mounted () {

        //console.log(this.authclients);

        SwaggerParser.validate(this.spec,(err, api)=> {
            if (err) {
                console.error(err);
                this.error = err;
            }
            else {

                  this.api = api;

                   console.log(api);

                    let pathKeys = Object.keys(api.paths);

                  if(api.tags && api.tags.length > 0){
                      this.tags = api.tags;


                        this.tags.forEach(tag => {
                            
                            let obj  = {};

                            Object.values(api.paths).map((resource,path)=>{


                                Object.values(resource).forEach((method,k) => {

                                    let mObj  = {};

                                    Object.keys(resource).forEach(m => {

                                        mObj[m] = method;

                                           
                                    });
                            
                                    if(method.tags.indexOf(tag.name) != -1){
                                        obj[pathKeys[path]] = mObj;
                                        
                                       
                                    }

                                });

                                 this.paths[tag.name] = obj;
                                  

                            });
                        });


                   
                  }
                  else{
                      this.paths['default'] = api.paths;
                  }
                 

            }
        });


    },
    computed: {
        isOpened (model) {
            return model.opened  = false;
        },
        isOpen () {
            return this.open 
        }
    },
    components: {
        request,
        model
    }
}
</script>

<style scoped lang="scss">
.api  {
    /* background-color: yellow; */



    .header {
        display: flex;
        align-items: center;
        padding: 10px 20px 10px 0px;
        cursor: pointer;
        transition: all .2s;
        border-bottom: 1px solid rgba(59,65,81,.3);
        font-size: 24px;
        margin: 0 0 5px;
        font-family: sans-serif;
        color: #3b4151;     
        flex-wrap: wrap;   


        .host {
            font-size: 14px;
            font-weight: 400;
            padding: 0px;
            font-family: sans-serif;
            color: #3b4151;
            flex: 1;                       
        }

        .title {
            font-weight: 700;
            margin-right: 15px;
        }

        .description {
            font-size: 14px;
            font-weight: 400;
            flex: 1;
            padding: 0px;
            font-family: sans-serif;
            color: #3b4151;
            text-align: right;             
        }

    }

    .select{
        border: 1px solid;
        background: transparent;
        padding: 0px 10px;

    }

    .table{

        .header{
            justify-content: space-between;
            
            label{
                font-size: 18px;
            }
            

            .left{
                text-align: left
            }
            .right{
                text-align: right;
            }
        }
        h3{
            margin: 15px 0px;
        }

        h4{
            margin: 10px 0px;
        }
    } 



    .definitions{
        border: 1px solid rgba(59,65,81,.3);
        margin-top: 70px;


        .header{
            padding: 10px;
        }



        .models{

                padding: 10px;



                .model{
                    border-color: rgba(59, 65, 81, 0.3);
                    background: rgba(0, 0, 0, 0.1);
                    padding: 10px 20px;
                    margin-bottom: 10px;
                    border-radius: 5px;

                   
                }
        }
    }
}
</style>
