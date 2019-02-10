<template>
    <div id="authModal" class="modal" v-bind:class="{ open: isOpen }">

       
        <div class="modal-content">
            
             <div class="modal-header">
                <h3>GET NEW ACCESS TOKEN</h3>
                <span class="close" @click.prevent="close" >&times;</span>
            </div>


            <form>
                <div class="form-group">
                    <label for="accessTokenURL">Access Token URL</label>
                    <input type="url" v-model="auth.accessTokenURL" required name="accessTokenURL" id="accessTokenURL">
                </div>
                <div class="form-group">
                    <label for="clientID">Client ID</label>
                    <input type="text" v-model="auth.clientID" required name="clientID" id="clientID">
                </div>
                <div class="form-group">
                    <label for="clientSecret">Client Secret</label>
                    <input type="text" v-model="auth.clientSecret" required name="clientSecret" id="clientSecret">
                </div>
                <div class="form-group">
                    <label for="scope">Scope</label>
                    <input type="text" v-model="auth.scope" name="scope" id="scope" placeholder="eg. read:org, write:org">
                </div>
                <div class="form-group">

                    <button type="submit" class="submit" @click.prevent="requestToken" >Request Token</button>

                </div>
                <div v-if="error" class="error">
                    {{error}}
                </div>

            </form>
        </div>

    </div>
</template>      

<script>
import axios from 'axios'


export default {
    props: {
        "client":{
            type: Object,
            default: () => { 
                return { } 
            }
        },
        "isOpen":{
            type:Boolean,
            default: () => { 
                return true
            }

        }
    },
    data () {
        return {
            error:"",
            token:"",
            auth:{
                accessTokenURL:"",
                clientID:this.client.clientId,
                clientSecret:this.client.clientSecret,
                scope:this.client.scope
            }
        }
    },
    methods: {

        async requestToken(){
                const call = axios.create()

                var self = this;

                call.interceptors.response.use((response) => {
                    return response;
                },  (error)=> {
                    
                    // Do something with response error
                    if (error.response.status === 401) {
                    }


                    return Promise.reject(error.response);
                });       

            let url = this.auth.accessTokenURL;

            let formData = {
                grant_type:"client_credentials",
            }

            if(this.auth.scope){
                formData.scope = this.auth.scope;
            }

            const config = {
                url,
                method: 'POST',
                headers: {
                    'accept': 'application/json',
                    'Content-Type': 'application/x-www-form-urlencoded'
                },
                params: formData,
                data:{},
                auth:{
                    username: this.auth.clientID,
                    password: this.auth.clientSecret
                }
            }


            try {
                let response = await call.request(config);
                console.log(response);

                this.$root.$emit('accessToken', response.data);


                this.token = "";
                this.close();
              
            } catch (e) {
                console.log(e);
                this.error = "Something went wrong!";
            }


        },
        close(){
            this.isOpen = false;
        }
       
    },
   
}
</script>

<style  scoped lang="scss">

.modal {
  display: none; /* Hidden by default */
  position: fixed; /* Stay in place */
  z-index: 1; /* Sit on top */
  left: 0;
  top: 0;
  width: 100%; /* Full width */
  height: 100%; /* Full height */
  overflow: auto; /* Enable scroll if needed */
  background-color: rgb(0,0,0); /* Fallback color */
  background-color: rgba(0,0,0,0.4); /* Black w/ opacity */
}

.modal.open{
    display: block;
}

.modal-header{
    border-bottom: 1px solid #888;
    margin-bottom: 15px;

    h3{
        font-size: 22px;
        padding-bottom: 10px;
    }
}

/* Modal Content/Box */
.modal-content {
  background-color: #fefefe;
  margin: 15% auto; /* 15% from the top and centered */
  padding: 30px;
  border: 1px solid #888;
  width: 80%; /* Could be more or less, depending on screen size */
  position: relative;
}

/* The Close Button */
.close {
  color: #aaa;
  position: absolute;
  right: 25px;
  top: 10px;
  font-size: 28px;
  font-weight: bold;
}

.close:hover,
.close:focus {
  color: black;
  text-decoration: none;
  cursor: pointer;
}

form {
    .form-group{
        margin-bottom: 20px;
        display: flex;

        label{
                display: inline-block;
                font-size: 12px;
                flex-grow: 1;
                flex-basis: 0;
                font-weight: 800;
        }
        input{
            padding: 10px;
            flex-grow: 4;
        }

        button.submit{
            padding: 10px 20px;
            background: transparent;
            border: 1px solid #000;
            font-size: 14px;


        }
    }

    .error{
        color: #ff0000c7;
        font-size: 16px;
    }
}

</style>        