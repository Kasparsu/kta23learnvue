<script setup>
import { computed, ref, watch } from 'vue';

// Game Vars
let cookies = ref(0);
let buildings = ref([
  { name: 'Cursor', price: 15, cps: 0.1, count: 0 },
  { name: 'Grandma', price: 100, cps: 1, count: 0 },
  { name: 'Farm', price: 1100, cps: 8, count: 0 },
]);

// CPS Kalkulatsioon
const cps = computed(() => {
  return buildings.value.reduce((total, building) => {
    return total + building.cps * building.count;
  }, 0);
});

//golden cookie system
let goldenCookies = ref([]);
let goldenBonusActive = ref(false);

// Upgrade System 
let upgrades = ref([
  {
    name: "Better Clicks",
    cost: 100,
    effect: () => { clickPower.value += 1; }, // Ensure `clickPower.value`
    bought: false,
    description: "Increases cookies per click by +1."
  },
  {
    name: "Faster Grandmas",
    cost: 500,
    effect: () => { buildings.value[1].cps *= 2; },
    bought: false,
    description: "Doubles Grandma's CPS."
  },
  {
    name: "Cheaper Buildings",
    cost: 1000,
    effect: () => { buildings.value.forEach(b => b.price = Math.ceil(b.price * 0.9)); },
    bought: false,
    description: "Reduces all building prices by 10%."
  }
]);

let clickPower = ref(1); // Default click power alguseks 1

// Laeb local storage
function loadGame() {
  let save = JSON.parse(localStorage.getItem('cookieClickerSave'));
  if (save) {
    cookies.value = save.cookies ?? 0;
    buildings.value = save.buildings ?? buildings.value;
    goldenBonusActive.value = save.goldenBonusActive ?? false;
    clickPower.value = save.clickPower ?? 1;
    
    // saved upgrades
    if (save.upgrades) {
      upgrades.value = save.upgrades.map(savedUpgrade => {
        let original = upgrades.value.find(u => u.name === savedUpgrade.name);
        return original ? { ...savedUpgrade, effect: original.effect } : savedUpgrade;
      });
    }
  }
}

// save func
function saveGame() {
  localStorage.setItem('cookieClickerSave', JSON.stringify({
    cookies: cookies.value,
    buildings: buildings.value,
    goldenBonusActive: goldenBonusActive.value,
    upgrades: upgrades.value,
    clickPower: clickPower.value
  }));
}

// autosave iga 10 sek
setInterval(saveGame, 10000);

//earn cookies over time
setInterval(() => {
  cookies.value += cps.value;
}, 1000);

// Click Cookie arvestab clickpower upgradega
function cookieClick() {
  cookies.value += clickPower.value;
  saveGame();
}

// buy building
function buyBuilding(building) {
  if (cookies.value >= building.price) {
    cookies.value -= building.price;
    building.count++;
    building.price += Math.ceil((building.price / 100) * 15);
    saveGame();
  }
}

// Golden Cookie surprise
function clickGoldenCookie(id) {
  // Remove the clicked golden cookie from the screen
  goldenCookies.value = goldenCookies.value.filter(cookie => cookie.id !== id);

  let randomEffect = Math.random(); // Random number for lottery effect

  if (randomEffect < 0.4) {
    // 🍪 Instant Cookie Bonus (40% chance)
    let bonusCookies = Math.floor(Math.random() * 400) + 100; // Random between 100-500
    cookies.value += bonusCookies;
    alert(`🎉 Golden Cookie! You win ${bonusCookies} bonus cookies! 🍪`);
  } else if (randomEffect < 0.7) {
    // 🚀 Double CPS for 30s (30% chance)
    goldenBonusActive.value = true;
    let originalCPS = cps.value;
    cps.value *= 2; // Double CPS
    alert("🚀 Golden Cookie! Your CPS X 2 for 30 seconds!");
    setTimeout(() => {
      cps.value = originalCPS;
      goldenBonusActive.value = false;
      saveGame();
    }, 30000);
  } else {
    // 🎰 Random Discount (30% chance)
    buildings.value.forEach(b => b.price = Math.ceil(b.price * 0.8)); // 20% cheaper
    alert("💰 Golden cookie! All buildings cost now 20% less!");
  }
  saveGame();
}

// Surprise spawn goldencookies
function spawnGoldenCookie() {
  let id = Date.now();
  goldenCookies.value.push({ id, x: Math.random() * 90, y: Math.random() * 90 });

  setTimeout(() => {
    goldenCookies.value = goldenCookies.value.filter(cookie => cookie.id !== id);
  }, 5000);

  setTimeout(spawnGoldenCookie, Math.random() * 60000 + 30000);
}

function buyUpgrade(upgrade) {
  if (cookies.value >= upgrade.cost && !upgrade.bought) {
    console.log("Upgrading:", upgrade); // Debugging log
    if (typeof upgrade.effect === 'function') {
      cookies.value -= upgrade.cost;
      upgrade.effect();  // Apply the upgrade
      upgrade.bought = true;
      saveGame();
      console.log("Upgrade successful.");
    } else {
      console.error("Error: upgrade.effect is not a function!", upgrade);
    }
  }
}

// start from beginning
function resetGame() {
  if (confirm("Are you sure you want to start again?")) {
    localStorage.removeItem('cookieClickerSave');
    cookies.value = 0;
    buildings.value = [
      { name: 'Cursor', price: 15, cps: 0.1, count: 0 },
      { name: 'Grandma', price: 100, cps: 1, count: 0 },
      { name: 'Farm', price: 1100, cps: 8, count: 0 },
    ];
    goldenCookies.value = [];
    goldenBonusActive.value = false;
    upgrades.value.forEach(upgrade => upgrade.bought = false); 
    saveGame();
    location.reload();
  }
}

loadGame();

// goldencookie spawn
setTimeout(spawnGoldenCookie, Math.random() * 60000 + 30000);
</script>

<template>
  <div class="container p-5">
    <!-- Main Cookie Clicker UI -->
    <div class="columns is-multiline">
      <!-- Cookie & Stats Section -->
      <div class="column is-3 box has-background-primary has-text-centered">
        <h1 class="title has-text-white">🍪 Cookies: {{ cookies.toFixed(1) }}</h1>
        <h2 class="subtitle has-text-white">CPS: {{ cps.toFixed(1) }}</h2>
        <h3 class="subtitle has-text-warning">Click Power: {{ clickPower }}</h3> 
        <figure @click="cookieClick" class="image is-128x128 m-auto">
          <img src="https://pngimg.com/uploads/cookie/cookie_PNG13656.png" class="clickable-cookie" />
        </figure>
      </div>

      <!-- Buildings Section -->
      <div class="column is-3 box has-background-warning">
        <h2 class="title is-5 has-text-centered">Buildings 🏗️</h2>
        <span class="is-size-7 has-text-dark has-text-centered pb-3">
          Buy buildings that generate more cookies per second!.
        </span>
        <div class="buttons is-flex is-flex-direction-column">
          <button 
            v-for="building in buildings" 
            :key="building.name" 
            :disabled="cookies < building.price" 
            @click="buyBuilding(building)"
            class="button is-info m-1"
          >
            {{ building.name }} - {{ building.price }} 🍪 ({{ building.count }})
          </button>
        </div>
      </div>

      <!-- Upgrades Section -->
      <div class="column is-3 box has-background-light">
        <h2 class="title is-5 has-text-black has-text-centered">Upgrades 🛠️</h2>
        <span class="is-size-7 has-text-dark has-text-centered pb-3">
          Buy upgrades that improve your cookie production.
        </span>
        <div class="buttons is-flex is-flex-direction-column">
          <button 
            v-for="upgrade in upgrades" 
            :key="upgrade.name" 
            :disabled="cookies < upgrade.cost || upgrade.bought" 
            @click="buyUpgrade(upgrade)" 
            class="button is-success m-1"
          >
            {{ upgrade.name }} ({{ upgrade.cost }} 🍪)
          </button>
        </div>
      </div>

      <!-- Reset Button -->
      <div class="column is-3 box has-background-danger has-text-centered">
        <button @click="resetGame" class="button is-dark is-medium">
          Reset Game 🔄
        </button>
      </div>
    </div>

    <!-- Golden Cookie Info Box -->
    <div class="golden-cookie-info">
      ⚠️ Watch out for Golden Cookies! Click them for a surprise reward: 
      <strong>Double CPS, Free Cookies, or Instant Building Discount!</strong>
    </div>

    <!-- Golden Cookie Display -->
    <div 
      v-for="cookie in goldenCookies" 
      :key="cookie.id" 
      @click="clickGoldenCookie(cookie.id)"
      class="golden-cookie" 
      :style="{ left: cookie.x + '%', top: cookie.y + '%' }"
    >
      ⭐
    </div>
  </div>
</template>