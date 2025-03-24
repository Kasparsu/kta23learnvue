<template>
    <div class="cookie-clicker">
      <!-- Main Cookie Display -->
      <div class="cookie-container">
        <h1>Cookies: {{ formattedCookies }}</h1>
        <h2>CPS: {{ cookiesPerSecond.toFixed(1) }}</h2>
        <button class="cookie" @click="increment">
          <img src="../assets/cookie.png" alt="Cookie" @error="handleCookieError"/>
        </button>
      </div>
  
      <!-- Upgrade Images Display Area -->
      <div class="upgrade-display-area">
      <div class="debug-info" v-if="debugMode">
        Debug Info: 
        Cursor: {{ availableUpgrades[0].owned }} |
        Grandma: {{ availableUpgrades[1].owned }} |
        Farm: {{ availableUpgrades[2].owned }}
      </div>

      <img 
        v-if="availableUpgrades[0].owned > 0"
        :src="cursorImage"
        class="upgrade-visual"
        :style="getUpgradeStyle(0)"
        @error="handleImageError('cursor')"
      >
      <img 
        v-if="availableUpgrades[1].owned > 0"
        :src="grandmaImage"
        class="upgrade-visual"
        :style="getUpgradeStyle(1)"
        @error="handleImageError('grandma')"
      >
      <img 
        v-if="availableUpgrades[2].owned > 0"
        :src="farmImage"
        class="upgrade-visual"
        :style="getUpgradeStyle(2)"
        @error="handleImageError('farm')"
      >
    </div>
      <!-- Upgrades Section -->
      <div class="upgrades-section">
        <h2>Upgrades</h2>
        <div class="upgrades-list">
          <div 
            v-for="(upgrade, index) in availableUpgrades" 
            :key="'upgrade-' + index"
            class="upgrade-card"
            :class="{ 
              'can-afford': cookies >= upgrade.cost,
              'purchased': upgrade.owned > 0
            }"
            @click="purchaseUpgrade(index)"
          >
            <div class="upgrade-header">
              <h3>{{ upgrade.name }}</h3>
              <span class="owned-count" v-if="upgrade.owned > 0">×{{ upgrade.owned }}</span>
            </div>
            <div class="upgrade-stats">
              <p class="cost">Cost: {{ formatNumber(upgrade.cost) }} cookies</p>
              <p class="value">+{{ upgrade.value }} CPS</p>
            </div>
            <div class="upgrade-description">
              <p>{{ upgrade.description }}</p>
            </div>
          </div>
        </div>
      </div>
  
      <!-- Achievement Notification -->
      <transition name="slide-fade">
        <div v-if="showAchievementNotification" class="achievement-notification">
          <span class="badge">✓</span>
          Achievement Unlocked: {{ achievementNotificationText }}
        </div>
      </transition>
    </div>
  </template>
  
  <script>
  export default {
    name: 'CookieClicker',
    data() {
      return {
        cookies: 0,
        totalClicks: 0,
        cookiesPerSecond: 0,
        availableUpgrades: [
          {
            name: "Cursor",
            description: "Automatic cookie clicking",
            cost: 25,
            value: 0.1,
            owned: 0
          },
          {
            name: "Grandma",
            description: "A nice grandma to bake more cookies",
            cost: 100,
            value: 0.5,
            owned: 0
          },
          {
            name: "Farm",
            description: "Grows cookie plants",
            cost: 1100,
            value: 4,
            owned: 0
          }
        ],
        showAchievementNotification: false,
        achievementNotificationText: "",
        cursorImage: require('../assets/upgrades/cursor.png'),
        grandmaImage: require('../assets/upgrades/grandma.png'),
        farmImage: require('../assets/upgrades/farm.png'),
        cookieImage: require('../assets/cookie.png'),
        imageErrors: {},
        debugMode: false
      }
    },
    computed: {
      formattedCookies() {
        return this.cookies.toFixed(1).replace(/\B(?=(\d{3})+(?!\d))/g, ",");
      }
    },
    mounted() {
      this.loadGame();
      this.startGameLoop();
    },
    beforeDestroy() {
      clearInterval(this.gameLoop);
    },
    methods: {
      increment() {
        this.cookies += 1;
        this.totalClicks++;
        this.checkAchievements();
      },
      getUpgradeStyle(index) {
    const positions = [
      { left: '15%', top: '20%' },
      { left: '35%', top: '30%' },
      { left: '75%', top: '25%' }
    ];
    return {
      ...positions[index],
      transform: `scale(${0.8 + this.availableUpgrades[index].owned * 0.1})`
    };
  },

  // Update your purchase method to ensure it's working
  purchaseUpgrade(index) {
    const upgrade = this.availableUpgrades[index];
    if (this.cookies >= upgrade.cost) {
      this.cookies -= upgrade.cost;
      upgrade.owned += 1; // Changed from ++ to += 1 for clarity
      this.cookiesPerSecond += upgrade.value;
      upgrade.cost = Math.floor(upgrade.cost * 1.15);
      this.saveGame(); // Ensure this is called
      
      // Force update (sometimes needed for Vue's reactivity)
      this.$forceUpdate(); 
      
      console.log(`Purchased ${upgrade.name}`, upgrade);
    }
  },
      },
      checkAchievements() {
        // Achievement checks would go here
      },
      startGameLoop() {
        this.gameLoop = setInterval(() => {
          this.cookies += this.cookiesPerSecond / 10;
          this.saveGame();
        }, 100);
      },
      formatNumber(num) {
        return num.toFixed(0).replace(/\B(?=(\d{3})+(?!\d))/g, ",");
      },
      saveGame() {
        const gameData = {
          cookies: this.cookies,
          upgrades: this.availableUpgrades,
          cps: this.cookiesPerSecond
        };
        localStorage.setItem('cookieClickerSave', JSON.stringify(gameData));
      },
      loadGame() {
        const savedData = localStorage.getItem('cookieClickerSave');
        if (savedData) {
          const gameData = JSON.parse(savedData);
          this.cookies = gameData.cookies || 0;
          this.availableUpgrades = gameData.upgrades || this.availableUpgrades;
          this.cookiesPerSecond = gameData.cps || 0;
        }
      },
      handleImageError(type) {
        const emojis = {
          cursor: '🖱️',
          grandma: '👵',
          farm: '🚜'
        };
        
        const svg = `<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100">
          <rect width="100" height="100" fill="#f0f0f0"/>
          <text x="50" y="60" font-size="60" text-anchor="middle">${emojis[type]}</text>
        </svg>`;
        
        const dataUri = `data:image/svg+xml;utf8,${encodeURIComponent(svg)}`;
        
        if (type === 'cursor') this.cursorImage = dataUri;
        if (type === 'grandma') this.grandmaImage = dataUri;
        if (type === 'farm') this.farmImage = dataUri;
      },
      handleCookieError() {
        this.cookieImage = 'data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100"><text x="50" y="60" font-size="60" text-anchor="middle">🍪</text></svg>';
      }
    }
  
  </script>
  
  <style scoped>
  .cookie-clicker {
    font-family: 'Arial', sans-serif;
    max-width: 800px;
    margin: 0 auto;
    padding: 20px;
    color: #333;
  }
  
  .cookie-container {
    text-align: center;
    margin-bottom: 30px;
  }
  
  .cookie {
    background: none;
    border: none;
    cursor: pointer;
    transition: transform 0.1s;
    padding: 0;
  }
  
  .cookie:active {
    transform: scale(0.95);
  }
  
  .cookie img {
    width: 200px;
    height: 200px;
    transition: all 0.2s;
  }
  
  .cookie:hover img {
    transform: scale(1.05);
  }
  
  .upgrade-display-area {
  height: 150px;
  background-color: #e6f7ff;
  border-radius: 10px;
  margin: 20px 0;
  position: relative;
  overflow: visible; /* Changed from hidden */
  border: 2px dashed #ccc; /* Temporary for debugging */
}

.upgrade-visual {
  position: absolute;
  width: 60px;
  height: 60px;
  transition: all 0.3s;
  z-index: 10;
}

.debug-info {
  position: absolute;
  top: 5px;
  left: 5px;
  background: rgba(0,0,0,0.7);
  color: white;
  padding: 5px;
  font-size: 12px;
  z-index: 100;
}
  
  .upgrades-section {
    background: #f8f8f8;
    padding: 20px;
    border-radius: 10px;
    margin-bottom: 20px;
  }
  
  .upgrades-list {
    display: grid;
    gap: 15px;
    grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  }
  
  .upgrade-card {
    background: white;
    border-radius: 8px;
    padding: 15px;
    cursor: pointer;
    transition: all 0.3s;
    border: 1px solid #e0e0e0;
  }
  
  .upgrade-card.can-afford:not(.purchased) {
    border-color: #4CAF50;
    background: #f0fff0;
  }
  
  .upgrade-card.can-afford:not(.purchased):hover {
    transform: translateY(-3px);
    box-shadow: 0 5px 15px rgba(0,0,0,0.1);
  }
  
  .upgrade-card.purchased {
    background: #f5f5f5;
  }
  
  .upgrade-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 10px;
  }
  
  .upgrade-header h3 {
    margin: 0;
    font-size: 1.2rem;
  }
  
  .owned-count {
    background: #4CAF50;
    color: white;
    padding: 2px 8px;
    border-radius: 10px;
    font-size: 0.8rem;
  }
  
  .upgrade-stats {
    display: flex;
    justify-content: space-between;
    margin-bottom: 10px;
    font-size: 0.9rem;
  }
  
  .cost {
    color: #e91e63;
    font-weight: bold;
  }
  
  .value {
    color: #4CAF50;
    font-weight: bold;
  }
  
  .upgrade-description {
    font-size: 0.85rem;
    color: #666;
  }
  
  .achievement-notification {
    position: fixed;
    top: 20px;
    left: 50%;
    transform: translateX(-50%);
    background: #4CAF50;
    color: white;
    padding: 12px 20px;
    border-radius: 5px;
    box-shadow: 0 3px 10px rgba(0,0,0,0.2);
    z-index: 100;
    display: flex;
    align-items: center;
  }
  
  .badge {
    background: white;
    color: #4CAF50;
    width: 20px;
    height: 20px;
    border-radius: 50%;
    display: inline-flex;
    align-items: center;
    justify-content: center;
    margin-right: 10px;
    font-weight: bold;
  }
  
  /* Animations */
  .fade-enter-active, .fade-leave-active {
    transition: opacity 0.5s, transform 0.5s;
  }
  .fade-enter, .fade-leave-to {
    opacity: 0;
    transform: scale(0.8);
  }
  
  .slide-fade-enter-active {
    transition: all 0.3s ease;
  }
  .slide-fade-leave-active {
    transition: all 0.3s cubic-bezier(1.0, 0.5, 0.8, 1.0);
  }
  .slide-fade-enter, .slide-fade-leave-to {
    transform: translateY(-20px);
    opacity: 0;
  }
  </style>