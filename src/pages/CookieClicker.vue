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
  let baseCPS = buildings.value.reduce((total, building) => {
    return total + building.cps * building.count;
  }, 0);
  
  // Apply golden cookie bonus if active
  return goldenBonusActive.value ? baseCPS * 2 : baseCPS;
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

let clickPower = ref(1); // Game Start click power is 1
let bonusTimeRemaining = ref(0); // Time remaining in seconds without bonus


// Load saved game


function loadGame() {
  let save = JSON.parse(localStorage.getItem('cookieClickerSave'));
  if (save) {
    cookies.value = save.cookies ?? 0;
    buildings.value = save.buildings ?? buildings.value;
    clickPower.value = save.clickPower ?? 1;
    
    // saved upgrades
    if (save.upgrades) {
      upgrades.value = save.upgrades.map(savedUpgrade => {
        let original = upgrades.value.find(u => u.name === savedUpgrade.name);
        return original ? { ...savedUpgrade, effect: original.effect } : savedUpgrade;
      });
    }
  }

  // Check if there's an active golden bonus
  const bonusExpiry = localStorage.getItem('goldenBonusExpiry');
  if (bonusExpiry) {
    const expiryTime = parseInt(bonusExpiry);
    if (expiryTime > Date.now()) {
      // Bonus is still active
      goldenBonusActive.value = true;
      setupBonusTimer(expiryTime);
    } else {
      // Bonus has run out while the game was closed
      localStorage.removeItem('goldenBonusExpiry');
      goldenBonusActive.value = false;
    }
  } else {
    goldenBonusActive.value = false;
  }
}

// SaveGame function
function saveGame() {
  localStorage.setItem('cookieClickerSave', JSON.stringify({
    cookies: cookies.value,
    buildings: buildings.value,
    upgrades: upgrades.value,
    clickPower: clickPower.value
  }));
}

// Autosave every 10 seconds
setInterval(saveGame, 10000);

//Earn cookies over time every second
setInterval(() => {
  cookies.value += cps.value;
}, 1000);

// Click Cookie function also works with clickPower
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
    
    // Store when the bonus should expire (30 seconds from now) lets give 3 second time for notitification of golden cookie
    const expiryTime = Date.now() + 33000;
    localStorage.setItem('goldenBonusExpiry', expiryTime.toString());
    alert("🚀 Golden Cookie! Your CPS X 2 for ≈ 30 seconds!");
    // Setup the timer to end the bonus
    setupBonusTimer(expiryTime);

  } else {
    // 🎰 Random Discount (30% chance)
    buildings.value.forEach(b => b.price = Math.ceil(b.price * 0.8)); // 20% cheaper
    alert("💰 Golden cookie! All buildings cost now 20% less!");
  }
  saveGame();
}

// Function to handle bonus timers
function setupBonusTimer(expiryTime) {
  const updateRemainingTime = () => {
    const now = Date.now();
    const remaining = Math.max(0, Math.floor((expiryTime - now) / 1000)); // convert to seconds
    
    bonusTimeRemaining.value = remaining;    
    if (remaining <= 0) {
      goldenBonusActive.value = false;
      localStorage.removeItem('goldenBonusExpiry');
      bonusTimeRemaining.value = 0;
      saveGame();
      clearInterval(timerInterval);
    }
  };
  
  // Initial update
  updateRemainingTime();  
  // update every second
  const timerInterval = setInterval(updateRemainingTime, 1000);
}


// Surprise spawn goldencookies
function spawnGoldenCookie() {
  let id = Date.now();
  goldenCookies.value.push({ id, x: Math.random() * 90, y: Math.random() * 90 });
  setTimeout(() => {
    goldenCookies.value = goldenCookies.value.filter(cookie => cookie.id !== id);
  }, 5000); // 5 seconds to click the cookie

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
    clickPower.value = 1;
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
      <div class="column cookie-clicker-column is-3 box has-background-primary has-text-centered">
        <h1 class="title has-text-white">🍪 Cookies: {{ cookies.toFixed(1) }}</h1>
        <h2 class="subtitle has-text-white">CPS: {{ cps.toFixed(1) }}</h2>
            <!-- bonus timer -->
        <h3 v-if="goldenBonusActive && bonusTimeRemaining > 0" class="subtitle has-text-warning">
  🚀 CPS Bonus: {{ bonusTimeRemaining }}s remaining
</h3>
        <h3 class="subtitle has-text-warning">Click Power: {{ clickPower }}</h3> 
        <figure @click="cookieClick" class="image is-128x128 m-auto">
          <img src="https://pngimg.com/uploads/cookie/cookie_PNG13656.png" class="clickable-cookie" />
        </figure>
      </div>

      <!-- Buildings Section -->
      <div class="column cookie-clicker-column is-3 box has-background-warning">
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
      <div class="column cookie-clicker-column is-3 box has-background-light">
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
      <div class="column cookie-clicker-column is-3 box has-background-danger has-text-centered">
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

<style scoped>
@import "https://cdn.jsdelivr.net/npm/bulma@1.0.2/css/bulma.min.css";

.golden-cookie {
  position: absolute;
  width: 60px;
  height: 60px;
  background: radial-gradient(circle, gold, orange);
  border-radius: 50%;
  font-size: 2rem;
  text-align: center;
  cursor: pointer;
  display: flex;
  justify-content: center;
  align-items: center;
  transition: transform 0.2s;
  box-shadow: 0px 0px 10px rgba(255, 215, 0, 0.8);
  z-index: 999;
}

.golden-cookie:hover {
  transform: scale(1.2);
}

.clickable-cookie {
  transition: transform 0.1s ease-in-out;
  cursor: pointer;
}

.clickable-cookie:active {
  transform: scale(0.8);
}

.golden-cookie-info {
  position: relative;
  top: 20px;
  left: 50%;
  transform: translateX(-50%);
  background-color: rgba(255, 215, 0, 0.9);
  color: black;
  font-size: 14px;
  font-weight: bold;
  padding: 10px 15px;
  border-radius: 8px;
  box-shadow: 0px 2px 5px rgba(0, 0, 0, 0.2);
  text-align: center;
  max-width: 400px;
  z-index: 999;
  border: 2px solid red;
}

/* scoped column styling only this component */
.cookie-clicker-column {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  min-height: 300px; 
}
</style>