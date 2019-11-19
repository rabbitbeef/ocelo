<template>
<div>
    <img :src="Image[Status]"/>
</div>
</template>

<script>
export default {
    props:{
        Status:{
            type:String,
            default:"",
        },
    },

    data:function(){
        return{
        Image:{
            "blank":"./"+require('../../img/blank.png'),
            "white":"./"+require('../../img/white.png'),
            "black":"./"+require('../../img/black.png'),
        },
        }
    }
}
</script>