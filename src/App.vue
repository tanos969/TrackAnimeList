<script setup>
import {ref, computed, onMounted} from 'vue';

const query = ref('');
const my_anime = ref([]);
const search_result = ref([]);

const my_anime_asc = computed(() => {
    return my_anime.value.sort((a, b)=> {
        return a.title.localeCompare(b.title);
    });
});


const searchAnime = () => {
    const url = `https://api.jikan.moe/v4/anime?q=${query.value}`;
    fetch(url)
        .then(res => res.json())
        .then( res => {
            console.log(res);
            search_result.value = res.data;
        });
};

const handleInput = e => {
    if (!e.target.value){
        search_result.value = [];
    }
};

const addAnime = anime => {
    search_result.value = [];
    query.value = '';

    my_anime.value.push({
        id: anime.mal_id,
        title: anime.title,
        image: anime.images.jpg.image_url,
        episodes: anime.episodes,
        watched: 0

    });

    localStorage.setItem('my_anime', JSON.stringify(my_anime.value));
};

const removeAnime = anime => {
    let index = my_anime.value.indexOf(anime);
    my_anime.value.splice(index, 1);
    localStorage.setItem('my_anime', JSON.stringify(my_anime.value));
}

const incresaseAnime = anime => {
    anime.watched++;
    localStorage.setItem('my_anime', JSON.stringify(my_anime.value));
};
const decresaseAnime = anime => {
    anime.watched--;
    localStorage.setItem('my_anime', JSON.stringify(my_anime.value));
};

onMounted(() => {
    my_anime.value = JSON.parse(localStorage.getItem('my_anime')) || [];
});

</script>

<template>
  <main>
    <h1>Anime tracker</h1>
    <form @submit.prevent="searchAnime">
        <input 
            type="text"
            placeholder="search for an anime ..."
            v-model="query"
            @input="handleInput"
        >
        <button type="submit" class="btn">Search</button>
    </form>
    <div class="results" v-if="search_result.length > 0">
        <div class="result" v-for="anime in search_result" v-bind:key="anime.mal_id">
            <img :src="anime.images.jpg.image_url">
            <div class="details">
                <h3>{{ anime.title }}</h3>
                <p :title="anime.synopsis" v-if="anime.synopsis">
                    {{ anime.synopsis.slice(0, 120) }} ...
                </p>
                <span class="flex-1"></span>
                <button class="btn" @click="addAnime(anime)">Add to my anime list</button>
            </div>
        </div>
    </div>
    <div class="myanime" v-if="my_anime.length > 0">
        <h2>My anime</h2>
        <div v-for="anime in my_anime_asc" class="anime" v-bind:key="anime.id">
            <img :src="anime.image">
            <h3>{{ anime.title }}</h3>
            <div class="flex-1"></div>
            <span class="episodes">
                {{ anime.watched }} / {{ anime.episodes }}
            </span>
            <button
                v-if="anime.episodes !== anime.watched"
                @click="incresaseAnime(anime)"
                class="btn"
            >+</button>
            <button
                v-if="anime.watched > 0"
                @click="decresaseAnime(anime)"
                class="btn"
            >-</button>
            <span class="remove" @click="removeAnime(anime)">x</span>
        </div>
    </div>

  </main>
</template>

<style scoped>
    *{
        margin: 0;
        padding: 0;
        box-sizing: border-box;
        font-family: 'Fira sans', sans-serif;
    }
    body {
        background-color: #EEE;
    }
    main {
        margin: 0 auto;
        max-width: 768px;
        padding: 1.5rem;
    }

    h1 {
        text-align: center;
        margin-bottom: 1.5rem;
    }
    form{
        display: flex;
    }
    form input {
        appearance: none;
        outline: none;
        border: none;
        background: white;
        
        display: block;
        color: #888;
        font-size: 1.125rem;
        padding: 0.5rem 1rem;
        width: 100%;
    }

    .btn {
        appearance: none;
        outline: none;
        border: none;
        cursor: pointer;
        background: none;
        display: block;
        padding: 0.5rem 1rem;
        background-image: linear-gradient(to right, deeppink 50%, darkviolet 50%);
        background-size: 200%;
        color: #FFF;
        font-size: 1.125rem;
        font-weight: 700;
        text-transform: uppercase;
        transition: 0.4s;
    }
    .btn:hover {
        background-position: right;
    }

.results {
    background-color: #FFF;
    border-radius: 0.5rem;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    max-height: 480px;
    overflow-y: scroll;
    margin-bottom: 1.5rem;
}
.result{
    display: flex;
    margin: 1rem;
    padding: 1rem;
    border: 1px solid #ccc;
    border-radius: 0.5rem;
    transition: 0.4s;
}
.result img {
    width: 100px;
    border-radius: 1rem;
    margin-right: 1rem;
}
.details {
    flex: 1 1 0%;
    display: flex;
    align-items: flex-start;
    flex-direction: column;
}
.flex-1{
    flex: 1 1 0%;
}
.details h3{
    font-size: 1.25rem;
    margin-bottom: 0.5rem;
}
.details p {
    font-size: 0.875rem;
    line-height: 1.4;
    margin-bottom: 0.5rem;
}
.details .btn{
    margin-left: auto;
}
.myanime h2{
    color: #888;
    font-weight: 400;
    margin-bottom: 1.5rem;
}
.myanime .anime{
    display: flex;
    align-items: center;
    margin-bottom: 1.5rem;
    background-color: #FFF;
    padding: 1rem;
    border-radius: 0.5rem;
    box-shadow: 0px 2px 4px rgba(0, 0, 0, 0.1);
    position: relative;
}
.anime img{
    width: 72px;
    height: 72px;
    object-fit: cover;
    border-radius: 1rem;
    margin-right: 1rem;
}
.anime h3 {
    color: #888;
    font-size: 1.125rem;
}
.anime .episodes{
    margin-right: 1rem;
    color: #888;
}
.anime .btn:first-of-type {
    margin-right: 1rem;
}
.anime .btn:last-of-type {
    margin-right: 0;
}
.anime .remove {
    float: right;
    padding: 0px 5px;
    background-image: linear-gradient(to right, deeppink 50%, darkviolet 50%);
    background-size: 200%;
    transition: 0.4s;
    color:#EEE;
    position: absolute;
    top: 0;
    right: 0;
    border-radius: 0.5rem;
    margin: 5px;
    cursor: pointer;
}
.anime .remove:hover {
    background-position: right;
}
</style>
