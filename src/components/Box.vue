<template>
    <div class="box">in a box</div>
</template>

<script>
export default {
  name: 'Box',
  props: {
    msg: String
  } 
}
</script>

<style scoped>
.box{
    width: 50px;
    height: 50px;
    background-color: green;
}
</style>