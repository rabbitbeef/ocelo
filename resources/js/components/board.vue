<template>
<div>
<table>
    <tr v-for="(y,yindex) in Board" :key='yindex'>
    <td v-for="(x,xindex) in y" :key='xindex' >
        <div @click="onClick(yindex,xindex)">
        <switch-box v-bind:isOn='Board[yindex][xindex]'></switch-box>
        </div>
    </td>
    </tr>
</table>
{{sumcheck}}<br>
<div v-show="clear">
    Congratuation!
</div>
</div>
</template>

<script>

import Box from "./box.vue";

export default {
    props:{
        N:{
            type:Number,
            default:4,
        }
    },

    data:function(){
        return {
            Board:Array,
            sumcheck:0,
            clear:false
        }
    },

    mounted:function(){
        this.Board = new Array(this.N);
        for(let i = 0; i < this.N; i++) {
            this.Board[i] = new Array(this.N);
            for(let j=0;j<this.N;j++){
                if(Math.floor(Math.random(0,1)*2)>=1){
                    this.Board[i][j] = true;
                    this.sumcheck++;
                }else{
                    this.Board[i][j] = false;
                }
                
            }
        }       
    },

    methods:{
        acheck:function(y,x){
            if(x>=this.N||y>=this.N||x<0||y<0){
                return;
            }
            this.Board[y][x] = !this.Board[y][x];

            if(this.Board[y][x]){
            this.sumcheck++;
            }
            else{
            this.sumcheck--;
            }
        },

        onClick:function(y,x){
            this.acheck(y-1,x);
            this.acheck(y+1,x);
            this.acheck(y,x+1);
            this.acheck(y,x-1);
            this.acheck(y,x);

            if(this.sumcheck===0){
                this.clear=true;
            }

        }
    },

    components:{
        Box,
    }
    
}
</script>