<script setup>
import { computed, ref, onMounted } from 'vue';

let cookies = ref(0);
let goldenCookies = ref(0);
let goldenCookieVisible = ref(false);
let level = ref(0);
let buildings = ref([
    { name: 'Cursor', price: 15, cps: 0.1, count: 0 },
    { name: 'Grandma', price: 100, cps: 1, count: 0 },
    { name: 'Farm', price: 1100, cps: 8, count: 0 },
]);

setInterval(() => {
    cookies.value += cps.value;
}, 1000);

setInterval(() => {
    if (Math.random() < 0.1) {
        goldenCookieVisible.value = true;
    }
}, 1000);

function cookieClick() {
    cookies.value += 1;
}

function buyBuilding(building) {
    cookies.value -= building.price;
    building.count++;
    building.price += Math.ceil((building.price / 100) * 15)
}

function buyGoldenUpgrade() {
    if (goldenCookies.value >= 5) {
        goldenCookies.value -= 5;
        cookies.value += 1000;
    }
}

function unlockSpecialBuilding() {
    if (goldenCookies.value >= 10) {
        goldenCookies.value -= 10;
        buildings.value.push({
            name: 'Portal',
            price: 100000,
            cps: 100,
            count: 0
        });
    }
}

const cps = computed(() => {
    return buildings.value.reduce((total, building) => {
        return total + building.cps * building.count;
    }, 0)
});

const nextLevelCost = computed(() => {
    return Math.pow(2, level.value) * 1000;
});

function levelUp() {
    if (cookies.value >= nextLevelCost.value) {
        cookies.value -= nextLevelCost.value;
        level.value++;

        buildings.value.forEach(building => {
            building.cps *= 1.1; 
        });

    }
}

function collectGoldenCookie() {
    goldenCookies.value += 1;
    goldenCookieVisible.value = false;  
}
function loadGame() {
    const savedCookies = localStorage.getItem('cookies');
    const savedGoldenCookies = localStorage.getItem('goldenCookies');
    const savedLevel = localStorage.getItem('level');
    const savedBuildings = localStorage.getItem('buildings');

    if (savedCookies !== null) {
        cookies.value = parseFloat(savedCookies);
    }
    if (savedGoldenCookies !== null) {
        goldenCookies.value = parseInt(savedGoldenCookies);
    }
    if (savedLevel !== null) {
        level.value = parseInt(savedLevel);
    }
    if (savedBuildings !== null) {
        buildings.value = JSON.parse(savedBuildings);
    }
}

function saveGame() {
    localStorage.setItem('cookies', cookies.value);
    localStorage.setItem('goldenCookies', goldenCookies.value);
    localStorage.setItem('level', level.value);
    localStorage.setItem('buildings', JSON.stringify(buildings.value));  
}
onMounted(() => {
    loadGame();  
});

setInterval(() => {
    saveGame();
}, 10000);  

</script>
<template>
    <div class="columns">
        <div class="column is-3 has-background-primary has-text-centered">
            <p class="is-size-1">Cookies: {{ +parseFloat(cookies).toFixed(1) }}</p> 
            <p class="is-size-5">CPS: {{ +parseFloat(cps).toFixed(1) }}</p>
            <p class="is-size-5">Golden Cookies: {{ goldenCookies }}</p>
            <p class="is-size-5">Level: {{ level }}</p>
            <figure @click="cookieClick" class="image is-1by1 m-5">
                <img src="https://attic.sh/ugg6gxrtlxlydd3n8ni73cksb7s0" />
            </figure>
        </div>

        <div v-if="goldenCookieVisible" class="golden-cookie-container">
            <figure @click="collectGoldenCookie" class="image is-1by1 m-5 golden-cookie">
                <img src="https://freepngdesign.com/content/uploads/images/cookie-2784424382.png" alt="Golden Cookie" width="75" height="75" />
            </figure>
            <p class="is-size-5">Click to collect!</p>
        </div>

        <div class="column has-background-info">
            <button v-if="goldenCookies >= 5" @click="buyGoldenUpgrade" class="button is-success is-large is-fullwidth">
                Buy Golden Upgrade (5 Golden Cookies)
            </button>
            <button v-if="goldenCookies >= 10" @click="unlockSpecialBuilding"
                class="button is-warning is-large is-fullwidth">
                Unlock Special Building (10 Golden Cookies)
            </button>
            <button v-if="cookies >= nextLevelCost" @click="levelUp" class="button is-danger is-large is-fullwidth">
                Level Up ({{ nextLevelCost }})
            </button>
        </div>
        <div class="column is-2 has-background-warning">
            <button v-for="building in buildings" :disabled="cookies < building.price" @click="buyBuilding(building)"
                class="button is-primary is-large is-fullwidth">{{ building.name }} {{ building.price }} {{
                    building.count }}</button>
        </div>
    </div>
</template>

<style scoped>
.golden-cookie-container {
    position: absolute;
    top: 50%;  
    left: 50%;
    transform: translateX(-50%);
    animation: slideUp 3s ease-in-out forwards;
}

@keyframes slideUp {
    0% {
        transform: translateY(100px) translateX(-50%);  
        opacity: 0;
    }
    50% {
        transform: translateY(-30px) translateX(-50%);  
        opacity: 0.7;
    }
    100% {
        transform: translateY(0) translateX(-50%);  
        opacity: 1;
    }
}

.golden-cookie img {
    width: 75px;  
    height: 75px;
}
</style>
