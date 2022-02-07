<template>
    <div class="todo-item">
        <div>{{props.dispText}}</div>
        <div>
            <button v-on:click="deleteClick">Delete</button>
        </div>
    </div>
</template>

<script>
import { defineComponent } from '@vue/composition-api'

export default defineComponent({
    setup() {
        
    },
    props:{
        props: Object
    },
    methods:{
        deleteClick(){
            console.log(this.props);
            this.$emit('delete', this.props.id)
        }
    }
})
</script>

<style scoped>

.todo-item{
    display: flex;
    flex-direction: row;
    column-gap: 1em;
}

</style>