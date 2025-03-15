<script setup>
import { computed, ref,onMounted } from 'vue';
import '../styles/CookieAnimation.css';

let cookies = ref(0);
let buildings = ref([
    {name: 'Cursor', price: 15, cps: 0.1, count: 0},
    {name: 'Grandma', price: 100, cps: 1, count: 0},
    {name: 'Farm', price: 1100, cps: 8, count: 0},
]);

//Raining cookies
let fallingCookies = ref([]);

onMounted(() => {
    setInterval(() => {
        fallingCookies.value.push({
            left: `${Math.random() * 100}%`, 
            animationDuration: `${4 + Math.random() * 3}s`, 
            animationDelay: `${Math.random() * 2}s`,
        });
    }, 550); 
});


setInterval(() => {
    cookies.value += cps.value;
}, 1000);

const press = ref(false);

function cookieClick(){

    cookies.value += 1;
     press.value = true
  setTimeout(() => {
    press.value = false
  }, 150)
    const figure = event.currentTarget.getBoundingClientRect();
    const x = event.clientX - figure.left;
    const y = event.clientY - figure.top;

    const miniCookie = document.createElement("img");
    miniCookie.src = "https://pngimg.com/uploads/cookie/cookie_PNG13656.png";
    miniCookie.classList.add("mini-cookie");  

    miniCookie.style.left = `${x - 10}px`; 
    miniCookie.style.top = `${y - 10}px`;   

    event.currentTarget.appendChild(miniCookie);

    miniCookie.addEventListener("animationend", () => {
        miniCookie.remove();
    });

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



</script>
<template>

    <div class="columns" id="CookieClicker">
        <div class="column is-3 has-background--bulma-primary-00 has-text-centered ">
            <div
            v-for="(cookie, index) in fallingCookies"
            :key="index"
            class="cookie-rain"
            :style="{
                left: cookie.left,
                animationDuration: cookie.animationDuration,
                animationDelay: cookie.animationDelay
            }"
        >
            <img src="https://pngimg.com/uploads/cookie/cookie_PNG13656.png" />
        </div>
            <div class="container  p-6">
            <p class="is-size-1">{{  +parseFloat(cookies).toFixed( 1 ) }} cookies</p>
            <p class="is-size-5">cookies per second: {{ +parseFloat(cps).toFixed( 1 ) }}</p>
            <figure @click="cookieClick" :class="{ press: press }" class="image is-1by1 m-5 cookie-container " >
                <img src="https://pngimg.com/uploads/cookie/cookie_PNG13656.png" draggable="false"/>
            </figure>
            </div>
        </div>

        <div class="column has-background-info">

        </div>
        <div class="column is-2 has-background-warning">
            <button v-for="building in buildings" :disabled="cookies < building.price" @click="buyBuilding(building)" class="button is-primary is-large is-fullwidth">{{ building.name }} {{ building.price }} {{ building.count }}</button>
        </div>
    </div>

</template>

<style scoped>
#CookieClicker {
  height: 94vh;
  overflow: hidden;}
</style>