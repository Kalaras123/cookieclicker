<script setup>
import { computed, ref, onMounted, onUnmounted } from 'vue';

let cookies = ref(0);
let mode = ref('buy');
let buildings = ref([
    {name: 'Cursor', basePrice: 15, price: 15, cps: 0.1, count: 0},
    {name: 'Grandma', basePrice: 100, price: 100, cps: 1, count: 0},
    {name: 'Farm', basePrice: 1100, price: 1100, cps: 8, count: 0},
    {name: 'Factory', basePrice: 12000, price: 12000, cps: 60, count: 0},
    {name: 'Bank', basePrice: 130000, price: 130000, cps: 400, count: 0},
    {name: 'Temple', basePrice: 1400000, price: 1400000, cps: 2500, count: 0},
    {name: 'Wizard Tower', basePrice: 20000000, price: 20000000, cps: 15000, count: 0}
]);
let gambleActive = ref(false);
let gambleStartCookies = ref(0);
let gambleMaxReward = ref(0);
let gambleCollected = ref(0);
let gambleCookieVisible = ref(false);
let gambleCookieX = ref(0);
let gambleCookieY = ref(0);
let gambleArea = ref(null);

let gambleInterval;
let gambleTimeout;
let gambleCookieTimeout;
let cookieInterval;

onMounted(() => {
    cookieInterval = setInterval(() => {
        cookies.value += cps.value;
    }, 1000);
})

onUnmounted(() => {
    clearInterval(cookieInterval);
    clearInterval(gambleInterval);
    clearTimeout(gambleTimeout);
    clearTimeout(gambleCookieTimeout);
})

function setMode(newMode){
    mode.value = newMode;
}

function cookieClick(){
    cookies.value += 1;
}

function buyBuilding(building){
    cookies.value -= building.price;
    building.count++;
    building.price = Math.ceil(building.basePrice * Math.pow(1.15, building.count))
}

function sellBuilding(building){
    if(building.count > 0){
        cookies.value += Math.floor(building.price / 4);
        building.count--;
        building.price = Math.ceil(building.basePrice * Math.pow(1.15, building.count))
    }
}

function formatNumber(value){
    if(value >= 1000000000){
        return (value / 1000000000).toFixed(1) + 'B';
    }
    if(value >= 1000000){
        return (value / 1000000).toFixed(1) + 'M';
    }
    if(value >= 1000){
        return (value / 1000).toFixed(1) + 'K';
    }
    return +parseFloat(value).toFixed(1);
}

const cps = computed(() => {
    return buildings.value.reduce((total, building) => {
        return total + building.cps * building.count;
    }, 0)
});

function startGamble(){
    if(gambleActive.value || cookies.value <= 0) return;

    gambleStartCookies.value = Math.floor(cookies.value);
    gambleMaxReward.value = gambleStartCookies.value * 2;
    gambleCollected.value = 0;
    cookies.value = 0;
    gambleActive.value = true;

    gambleInterval = setInterval(() => {
        spawnGambleCookie();
    }, 1000);

    gambleTimeout = setTimeout(() => {
        endGamble();
    }, 20000);
}

function endGamble(){
    gambleActive.value = false;
    gambleCookieVisible.value = false;

    clearInterval(gambleInterval);
    clearTimeout(gambleTimeout);
    clearTimeout(gambleCookieTimeout);
}

function spawnGambleCookie(){
    if(!gambleActive.value) return;
    if(gambleCollected.value >= gambleMaxReward.value) return;

    const area = gambleArea.value;
    if(!area) return;

    const areaWidth = area.clientWidth;
    const areaHeight = area.clientHeight;
    const cookieSize = 40;

    const maxX = areaWidth - cookieSize;
    const maxY = areaHeight - cookieSize;

    gambleCookieX.value = Math.floor(Math.random() * maxX);
    gambleCookieY.value = Math.floor(Math.random() * maxY);

    gambleCookieVisible.value = true;

    clearTimeout(gambleCookieTimeout);
    gambleCookieTimeout = setTimeout(() => {
        gambleCookieVisible.value = false;
    }, 2000);
}

function collectGambleCookie(){
    if(!gambleActive.value || !gambleCookieVisible.value) return;

    let reward = Math.max(1, Math.floor(gambleStartCookies.value / 8));
    let remainingReward = gambleMaxReward.value - gambleCollected.value;

    if(reward > remainingReward){
        reward = remainingReward;
    }

    cookies.value += reward;
    gambleCollected.value += reward;
    gambleCookieVisible.value = false;

    clearTimeout(gambleCookieTimeout);

    if(gambleCollected.value >= gambleMaxReward.value){
        endGamble();
    }
}

</script>
<template>
    <div class="columns">
        <div class="column is-3 has-background-primary has-text-centered has-text-light">
            <p class="is-size-1">Cookies: {{  formatNumber(cookies) }}</p>
            <p class="is-size-5">CPS: {{ formatNumber(cps) }}</p>
            <figure @click="cookieClick" class="image is-1by1 m-5">
                <img 
                    style="border-radius: 50%; cursor: pointer"
                    src="https://pngimg.com/uploads/cookie/cookie_PNG13656.png" 
                />
            </figure>
        </div>
        <div ref="gambleArea" class="column has-background-info" style="position: relative; min-height: 500px;">
            <div v-if="!gambleActive" class="is-flex is-justify-content-center is-align-items-center" style="height: 100%;">
                <button
                    @click="startGamble"
                    :disabled="cookies <= 0"
                    class="button is-danger is-large">
                    Gamble
                </button>
            </div>
            <img
                v-if="gambleActive && gambleCookieVisible"
                @click="collectGambleCookie"
                src="https://pngimg.com/uploads/cookie/cookie_PNG13656.png"
                :style="{
                    position: 'absolute',
                    left: gambleCookieX + 'px',
                    top: gambleCookieY + 'px',
                    width: '40px',
                    borderRadius: '50%',
                    cursor: 'pointer'
                }"
            />
        </div>
        <div class="column is-2 has-background-warning">
            <div class="buttons is-centered">
                <button 
                    @click="setMode('buy')"
                    :class="mode === 'buy' ? 'button is-primary' : 'button is-light'">
                    Buy
                </button>
                <button 
                    @click="setMode('sell')"
                    :class="mode === 'sell' ? 'button is-primary' : 'button is-light'">
                    Sell
                </button>
            </div>
            <button 
                v-for="building in buildings"
                :disabled="mode === 'buy' ? cookies < building.price : building.count === 0"
                @click="mode === 'buy' ? buyBuilding(building) : sellBuilding(building)"
                class="button is-primary is-large is-fullwidth">
                {{ building.name }}
                {{ mode === 'buy' ? formatNumber(building.price) : formatNumber(Math.floor(building.price / 4)) }}
                {{ building.count }}
            </button>
        </div>
    </div>
</template>