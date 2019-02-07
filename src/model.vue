<template>
    <div>
         <h3  @click="toggle">{{name}}</h3>
        <table v-show="isActive">
            <tr> 
                <th class="vtop">Description</th>
                <td>{{model.description}}</td>
            </tr>
            <tr v-bind:key="key" v-for="(prop, key) in model.properties">
                <th class="vtop">Body</th>
                <td><pre>{{(prop)}}</pre></td>
            </tr>
        </table>
    </div>
</template>


<script>
export default {
    props: ["model","name"],
    data () {
        return {
            isActive: false
        }
    },
    methods: {
       toggle() {
        this.isActive = !this.isActive;
        }
    },
   
}
</script>

<style  scoped lang="scss">

    pre {
        overflow-x: auto;
        white-space: pre-wrap;
        white-space: -moz-pre-wrap;
        white-space: -pre-wrap;
        white-space: -o-pre-wrap;
        word-wrap: break-word;
        background: #482f2fb0;
        color: #fff;
        padding: 10px;
        width: 100%;
    }


    .vtop{
        vertical-align: top;
    }


    table {
        border-collapse: collapse;
    }

    td {
        padding-bottom: 1em;
        padding-left: 1em;
    }

    h3{
        padding: 15px 0px;
        margin: 0px;
        cursor: pointer;
    }
    
</style>
