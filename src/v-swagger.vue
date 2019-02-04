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

            <label for="scheme">Schemes</label>
            <select v-model="scheme" id="scheme">
                <option v-for="(scheme, index) in api.schemes" :key="index" :value="scheme">{{scheme}}</option>
            </select>

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

        </section>


    </div>
</template>

<script>

import request from './request.vue'

var SwaggerParser = require('swagger-parser');

export default {
    props: [ 'spec' ],
    data () {

        return {
            scheme:'https',
            specInfo: this.spec,
            open: this.spec.opened || true,
            api:'',
            error: undefined,
            tags:[{
                name:'default',
                description:''
            }],
            paths:[]
        }
    },
    mounted () {

        SwaggerParser.validate(this.spec,(err, api)=> {
            if (err) {
                console.error(err);
                this.error = err;
            }
            else {

                  this.api = api;

                  // console.log(api);

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
        isOpen () {
            return this.open 
        }
    },
    components: {
        request
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


        .host {
            font-size: 14px;
            font-weight: 400;
            padding: 0 10px;
            font-family: sans-serif;
            color: #3b4151;
            flex: 1;                       
        }

        .title {
            font-weight: 700;
        }

        .description {
            font-size: 14px;
            font-weight: 400;
            flex: 1;
            padding: 0 10px;
            font-family: sans-serif;
            color: #3b4151;
            text-align: right;             
        }

    }

    .table{
        h3{
            margin: 15px 0px;
        }

        h4{
            margin: 10px 0px;
        }
    } 
}
</style>
