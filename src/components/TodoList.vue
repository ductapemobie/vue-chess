<template>
    <div>
        <div>Todo List</div>
        <button v-on:click="btnClick">Add Item</button>
        <button v-on:click="updateList">Update List</button>
        <div>
            <input v-model="textValue" placeholder="What to do"/>
        </div>
        <div class="list-div">
                <div v-for="item in listItems" :key="item.id">
                    <TodoItem :props="item" @delete="deleteItem"/>
                </div>
        </div>
    </div>
</template>

<script>
import { defineComponent } from '@vue/composition-api';
import TodoItem from './TodoItem.vue';

export default defineComponent({
    components:{
        TodoItem
    },
    props:{
        user: {
            type: String,
            requried: true,
        }
    },
    setup(props) {
        console.log(props);
    },
    data(){
        return {
            listItems:[],
            textValue:"",
            elementId: 0
        }
    },
    methods:{
        addItem(item){
            console.log(item);
            if (item){
                this.listItems.push({
                    id: this.elementId++,
                    dispText: item,
                });
            }
        },
        btnClick(){
            this.addItem(this.textValue);
        },
        updateList(){
            for (let i = 0; i < this.listItems.length; i++){
                this.listItems[i].dispText = "updated"
            }
        },
        deleteItem(id){
            console.log(id);
            this.listItems = this.listItems.filter(item => item.id != id);
        }
    }
})
</script>

<style scoped>

.list-div{
    width: max-content;
    margin: auto;
}

</style>