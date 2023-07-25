<template>
    <div>
        <h1 class="font-bold text-gray-800">List of movies</h1>

        <ul v-if="movies.data">
            <li v-for="movie in movies.data">
                <router-link :to="{ name: 'MovieDetails', params: { movieName: movie.name } }">
                    {{ movie.title }}
                </router-link>
            </li>
        </ul>
    </div>
</template>

<script setup>
import { createListResource } from 'frappe-ui';

const movies = createListResource({
    doctype: 'Movie',
    fields: ['name', 'title', 'release_date', 'director',],
    auto: true,
});
</script>