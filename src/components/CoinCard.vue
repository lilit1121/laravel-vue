<template>
  <div>
    <h2>Top 5 Coins by Market Cap</h2>
    <div class="coin-selection">
      <label>Sell:</label>
      <select v-model="fromCoin">
        <option v-for="c in coins" :value="c.id" :key="c.id">{{ c.name }}</option>
      </select>
      <input type="number" v-model.number="sellAmount" :max="balances[fromCoin]" :min="0" />
      <label>Buy:</label>
      <select v-model="toCoin">
        <option v-for="c in coins" :value="c.id" :disabled="c.id === fromCoin" :key="c.id">{{ c.name }}</option>
      </select>
      <button @click="sellCoinToCoin">Sell</button>
    </div>
    <table>
      <thead>
      <tr>
        <th>Coin</th>
        <th>Price</th>
        <th>Balance</th>
      </tr>
      </thead>
      <tbody>
      <tr v-for="coin in coins" :key="coin.id">
        <td>{{ coin.name }}</td>
        <td>{{ coin.current_price }}</td>
        <td v-if="coin.id === 'bitcoin'">{{ balances.bitcoin }}</td>
        <td v-else>{{ balances[coin.id] }}</td>
      </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
import { reactive, computed, onMounted, ref } from 'vue'
import axios from "axios";

export default {
  setup() {
    const coins = reactive([])
    const balances = computed(() => {
      const result = {}
      for (const coin of coins) {
        if (coin.id === 'bitcoin') {
          result[coin.id] = 5
        } else {
          result[coin.id] = 0
        }
      }
      return result
    })
    const fromCoin = ref('bitcoin')
    const toCoin = ref('ethereum')
    const sellAmount = ref(0)

    const fetchCoins = async () => {
      const response = await axios.get('https://api.coingecko.com/api/v3/coins/markets', {
        params: {
          vs_currency: 'usd',
          order: 'market_cap_desc',
          per_page: 5,
        }
      })
      coins.splice(0, coins.length, ...response.data)
    }

    const sellCoinToCoin = () => {
      if (sellAmount.value <= 0) {
        alert('Invalid amount')
        return
      }

      const fromCoinPrice = coins.find(c => c.id === fromCoin.value).current_price
      const toCoinPrice = coins.find(c => c.id === toCoin.value).current_price
      balances.value[fromCoin.value] -= sellAmount.value
      const toCoinAmount = sellAmount.value * 2 * toCoinPrice / fromCoinPrice
      balances.value[toCoin.value] += toCoinAmount
      sellAmount.value = 0
    }

    onMounted(() => {
      fetchCoins()
    })

    return {
      coins,
      balances,
      fromCoin,
      toCoin,
      sellAmount,
      sellCoinToCoin,
    }
  }
}
</script>


<style scoped>
.coin-selection {
  margin-bottom: 20px;
}
.coin-selection label {
  margin-right: 10px;
}
.coin-selection select, .coin-selection input[type=number] {
  margin-right: 20px;
}
</style>
