<script setup>
import { computed, ref } from 'vue';

const cookies = ref(0);

const upgrades = ref([
    {name: '2x Click', price: 100, effect: 'click', multiplier: 2, purchased: false},
    {name: '2x Cursor', price: 500, effect: 'cursor', multiplier: 2, purchased: false},
    {name: '3x Click', price: 1000, effect: 'click', multiplier: 3, purchased: false},
    {name: '3x Cursor', price: 3000, effect: 'cursor', multiplier: 3, purchased: false},
]);

const buildings = ref([
    {name: 'Cursor', price: 15, cps: 0.1, count: 0},
    {name: 'Grandma', price: 100, cps: 1, count: 0},
    {name: 'Farm', price: 1100, cps: 8, count: 0},
]);

setInterval(() => {
    cookies.value += cps.value;
}, 1000);

function cookieClick(){
    cookies.value += clickPower.value;
}

function buyBuilding(building){
    cookies.value -= building.price;
    building.count++;
    building.price += Math.ceil((building.price / 100) * 15)
}

const cps = computed(() => {
    return buildings.value.reduce((total, building) => {
        let buildingCPS = building.cps;
        if (building.name === 'Cursor') {
            buildingCPS *= cursorMultiplier.value; 
        }
        return total + buildingCPS * building.count;
    }, 0)
});

function buyUpgrade(upgrade) {
    if (cookies.value >= upgrade.price && !upgrade.purchased) {
        cookies.value -= upgrade.price;
        upgrade.purchased = true;
    }
}

const clickPower = computed(() => {
    return upgrades.value
        .filter(upg => upg.purchased && upg.effect === 'click')
        .reduce((total, upg) => total * upg.multiplier, 1);
});

const cursorMultiplier = computed(() => {
    return upgrades.value
        .filter(upg => upg.purchased && upg.effect === 'cursor')
        .reduce((total, upg) => total * upg.multiplier, 1);
});

</script>
<template>
    <div class="columns">
        <div class="column is-3 has-background-primary has-text-centered">

            <p class="is-size-1">Cookies: {{  +parseFloat(cookies).toFixed( 1 ) }}</p>
            <p class="is-size-5">CPS: {{ +parseFloat(cps).toFixed( 1 ) }}</p>
            <p class="is-size-5">Click Power: x{{ clickPower }}</p>
            <p class="is-size-5">Cursor Multiplier: x{{ cursorMultiplier }}</p>
            <figure @click="cookieClick" class="image is-1by1 m-5">
                <img src="https://pngimg.com/uploads/cookie/cookie_PNG13656.png" />
            </figure>
        </div>
        <div class="column has-background-info">
        </div>

 <!-- Store & Buildings Section -->
 <div class="column is-2">
            <!-- Store (Upgrades) -->
            <div class="card has-background-warning-light p-4 mb-4">
                <p class="is-size-5 has-text-centered">Store</p>
                <div v-for="upgrade in upgrades" :key="upgrade.name" class="card m-2 p-2">
                    <button 
                        :disabled="cookies < upgrade.price || upgrade.purchased"
                        @click="buyUpgrade(upgrade)"
                        class="button is-info is-fullwidth is-small">
                        {{ upgrade.name }} - Price: {{ upgrade.price }}
                        <span v-if="upgrade.purchased"> ✔</span>
                    </button>
                    <p class="is-size-7">Effect: {{ upgrade.effect === 'click' ? 'Click Power' : 'Cursor Efficiency' }} x{{ upgrade.multiplier }}</p>
                </div>
            </div>

            <!-- Buildings -->
            <div class="card has-background-info-light p-4">
                <p class="is-size-5 has-text-centered">Buildings</p>
                <div v-for="building in buildings" :key="building.name" class="card m-2 p-2">
                    <button 
                        :disabled="cookies < building.price"
                        @click="buyBuilding(building)"
                        class="button is-primary is-fullwidth is-small">
                        {{ building.name }} - Price: {{ building.price }} - Count: {{ building.count }}
                    </button>
                    <p class="is-size-7">CPS: {{ building.cps }}</p>
                </div>
            </div>
        </div>
    </div>
</template>