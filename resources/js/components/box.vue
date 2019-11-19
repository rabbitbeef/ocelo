<template>
    <input class=switchbox__checkbox type=checkbox v-bind:checked="isOn"/>
</template>

<script>
export default {
    props:{
        isOn:{
            type:Boolean,
            default:false,
        }
    }
}
</script>