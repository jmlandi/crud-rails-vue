<template>
    <h1>To-do List</h1>
    <p>Create and manage tasks to do.</p>
    
    <!-- Form -->
    <div id="todo-form">
        <input
            type="text"
            v-model="title"
            placeholder="Name your task"
            class="title-input"
        >
        <input
            type="text"
            v-model="body"
            placeholder="Describe your task"
            class="body-input"
        >
        <div class="button-row">
            <button v-if="isEditing" @click="updatePost()" class="update">Update</button>
            <button v-if="isEditing" @click="cancelEditing()">Cancel</button>
            <button v-else @click="createPost()">Create</button>
        </div>
            
    </div>
    
    <!-- Tasks -->
     <h2>Your Tasks</h2>
    <div v-for="post in posts" :key="post.id">
        <div class="task">  
            <h3 class="task-title">{{ post.title }}</h3>
            <p><span class="task-about">About your task:</span><br> {{ post.body }}</p>
            <div class="button-row">
                <button class="edit" @click="editPost(post.id)">Edit</button>
                <button class="remove" @click="removePost(post.id)">Remove</button>
            </div>
        </div>
    </div>
</template>

<script setup>
    import { ref, onMounted } from 'vue'; 

    const posts = ref([]);
    const title = ref('');
    const body = ref('');
    const postId = ref(0);
    const isEditing = ref(false);
    const API_URL = 'http://localhost:3000/posts';

    onMounted( async () => {
        const res = await fetch(API_URL);
        posts.value = await res.json();
    })

    const createPost = async () => {
        const res = await fetch(API_URL, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json' 
            },
            body: JSON.stringify({
                title: title.value,
                body: body.value
            })
        })
        const data = await res.json();
        posts.value.push(data);
        title.value = '';
        body.value = '';
        postId.value = 0;
    }
    
    const updatePost = async () => {
        const res = await fetch(`${API_URL}/${postId.value}`, {
            method: 'PUT',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({
                id: postId.value,
                title: title.value,
                body: body.value
            })
        })

        const data = await res.json();
        const index = posts.value.findIndex(post => post.id === data.id);
        posts.value[index] = data;

        title.value = '';
        body.value = '';
        postId.value = 0;
        isEditing.value = false;
    }

    const removePost = async (id) => {
        await fetch(`${API_URL}/${id}`, {
            method: 'DELETE'
        })
        posts.value = posts.value.filter(post => post.id !== id);
    }

    const editPost = async (id) => {
        const post = posts.value.find(post => post.id === id);
        
        title.value = post.title;
        body.value = post.body;
        postId.value = post.id;
        isEditing.value = true;

        window.scrollTo({
            top: 0,
            behavior: 'smooth'
        })
    }

    const cancelEditing = () => {
        title.value = '';
        body.value = '';
        postId.value = 0;
        isEditing.value = false;
    }
</script>

<style scoped>

    * {
        margin: 0px;
        padding: 0px;
        box-sizing: border-box;
    }

    button {
        width: 100px;
        height: 30px;
    }
    
    #todo-form {
        background-color: rgb(65, 65, 65);
        padding: 30px;
        display: flex;
        justify-content: center;
        align-items: center;
        flex-direction: column;
        gap: 15px;
        margin: 50px;
        border-radius: 15px;
        border: 1px solid #fff;
    }

    .title-input, .body-input {
        padding: 10px 20px;
        border-radius: 10px;
        border: none;
        width: 300px;
    }

    .task {
        background-color: rgb(65, 65, 65);
        padding: 15px;
        display: flex;
        justify-content: center;
        align-items: center;
        flex-direction: column;
        gap: 5px;
        margin: 50px;
        border-radius: 15px;
        border: 1px solid #fff;
    }

    .button-row {
        margin: 10px;
        display: flex;
        justify-content: center;
        align-items: center;
        flex-direction: row;
        gap: 10px;
    }

    .remove {
        background-color: rgb(171, 26, 0);
    }

    .update {
        background-color: rgb(0, 78, 0);
    }

    .task-title {
        border-bottom: 1px solid #676767;
        padding: 5px;
        margin: 10px;
    }

    .task-about {
        font-weight: 500;
        color: #676767;
        font-size: 13px;
    }
 
</style>