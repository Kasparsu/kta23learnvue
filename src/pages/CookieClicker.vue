<script setup>
import { computed, ref } from 'vue';

let cookies = ref(0);
let buildings = ref([
    {name: 'Cursor', price: 15, cps: 0.1, count: 0},
    {name: 'Grandma', price: 100, cps: 1, count: 0},
    {name: 'Farm', price: 1100, cps: 8, count: 0},
]);

setInterval(() => {
    cookies.value += cps.value;
}, 1000);

function cookieClick(){
    cookies.value += 1;
}

function buyBuilding(building){
    cookies.value -= building.price;
    building.count++;
    building.price += Math.ceil((building.price / 100) * 15)
}

const cps = computed(() => {
    return buildings.value.reduce((total, building) => {
        return total + building.cps * building.count;
    }, 0)
});

// some cool changes
// more stuff

</script>
<template>
    <div class="columns">
        <div class="column is-3 has-background-primary has-text-centered">
            <p class="is-size-1">Cookies: {{  +parseFloat(cookies).toFixed( 1 ) }}</p>
            <p class="is-size-5">CPS: {{ +parseFloat(cps).toFixed( 1 ) }}</p>
            <figure @click="cookieClick" class="image is-1by1 m-5">
                <img src="https://pngimg.com/uploads/cookie/cookie_PNG13656.png" />
            </figure>
        </div>
        <div class="column has-background-info">

        </div>
        <div class="column is-2 has-background-warning">
            <button v-for="building in buildings" :disabled="cookies < building.price" @click="buyBuilding(building)" class="button is-primary is-large is-fullwidth">{{ building.name }} {{ building.price }} {{ building.count }}</button>
        </div>
    </div>
</template>