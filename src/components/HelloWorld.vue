<template>
  <div class="estimate-tool">
    <h1 class="font-bold">託送・発電管理システム見積ツール</h1>
    <label>プラン選択</label>
    <ul>
      <li>
        <select v-model="selectedPlan" id="plan-select">
          <option v-for="(plan, index) in plans" :key="index" :value="plan">
            {{ plan.name }} - {{ plan.basePrice }} 円
          </option>
        </select>
      </li>
    </ul>

    <div class="customer-count">
      <label>
        需要家数入力
        <ul>
          <li>
            <input type="number" v-model="customerCount" min="0" />
          </li>
        </ul>
      </label>
    </div>

    <h3>オプションサービス料金</h3>
    <div v-for="(product, index) in products" :key="index" class="product">
      <label>
        <input type="checkbox" v-model="product.selected" />
        {{ product.name }} ({{ product.price }} 円)
      </label>
    </div>

    <div class="total">
      <h2>見積合計金額: {{ totalPrice }} 円</h2>
    </div>
    <button @click="generateEstimate">見積詳細</button>
    <div v-if="estimateGenerated">
      <h3>見積詳細</h3>
      <ul>
        <li v-for="(plan, index) in plans" :key="index">
          {{ plan.name }} - {{ plan.basePrice }} 円
        </li>
        <li v-for="(product, index) in selectedProducts" :key="index">
          {{ product.name }} - {{ product.price }} 円
        </li>
      </ul>
      <h4>合計金額: {{ totalPrice }} 円</h4>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from "vue";

//プラン情報
const plans = ref([
  { name: "ベーシック プラン", basePrice: 32000 },
  { name: "スタンダード プラン", basePrice: 40000 },
  { name: "コンプリート プラン", basePrice: 50000 },
]);

// 選択したプラン及び需要家数
const selectedPlan = ref(null);
const customerCount = ref(0);

// 需要家数による月額の計算
const monthlyFee = computed(() => {
  if (!selectedPlan.value) return 0;
  const basePrice = selectedPlan.value.basePrice;
  const baseCustomerLimit = 5000;
  const upperCustomerLimit = 75000;
  const generalPerCustomerFee = 5000;
  const basicSpecialFee = 3000;
  const basicSpecialFeeLimit = 2000;

  let totalFee = basePrice;

  if (
    customerCount.value > baseCustomerLimit &&
    customerCount.value <= upperCustomerLimit
  ) {
    const additionalCustomers = customerCount.value - baseCustomerLimit;

    if (selectedPlan.value.name === "スタンダード プラン") {
      totalFee += Math.ceil(additionalCustomers / 5000) * generalPerCustomerFee;
    } else if (selectedPlan.value.name === "コンプリート プラン") {
      totalFee += Math.ceil(additionalCustomers / 5000) * generalPerCustomerFee;
    } else if (selectedPlan.value.name === "ベーシック プラン") {
      if (customerCount.value <= 10000) {
        totalFee += Math.ceil(additionalCustomers / 5000) * basicSpecialFee;
      } else {
        const firstTierAdditionalFee =
          Math.ceil((10000 - baseCustomerLimit) / 5000) * basicSpecialFee;
        const remainingAdditionalFee =
          additionalCustomers - (10000 - baseCustomerLimit);
        const secondTierAdditionalFee =
          Math.ceil(remainingAdditionalFee / 5000) * generalPerCustomerFee;
        totalFee += firstTierAdditionalFee + secondTierAdditionalFee;
      }
    }
  } else if (customerCount.value > upperCustomerLimit) {
    if (selectedPlan.value.name === "ベーシック プラン") {
      totalFee =
        basePrice +
        Math.ceil((upperCustomerLimit - baseCustomerLimit) / 5000) *
          generalPerCustomerFee -
        basicSpecialFeeLimit;
    } else {
      totalFee =
        basePrice +
        Math.ceil((upperCustomerLimit - baseCustomerLimit) / 5000) *
          generalPerCustomerFee;
    }
  }

  return totalFee;
});

// オプションサービス
const products = ref([
  { name: "パスワード自動解除機能", price: 5000, selected: false },
  { name: "確定値、速報値自動取得機能", price: 5000, selected: false },
  { name: "30分速報値自動取得機能", price: 25000, selected: false },
  { name: "発電関連帳票自動取得機能", price: 5000, selected: false },
  { name: "再エネ卸帳票自動取得機能", price: 10000, selected: false },
  { name: "沖縄関連帳票自動取得機能", price: 10000, selected: false },
  { name: "全銀データ自動作成機能", price: 5000, selected: false },
]);

// オプションサービス総額計算
const optionsTotalPrice = computed(() => {
  return products.value
    .filter((product) => product.selected)
    .reduce((total, product) => total + product.price, 0);
});

const totalPrice = computed(() => {
  return monthlyFee.value + optionsTotalPrice.value;
});

// 선택된 제품 목록
const selectedProducts = computed(() => {
  return products.value.filter((product) => product.selected);
});

// 견적 생성 버튼 클릭 시 처리
const estimateGenerated = ref(false);
const generateEstimate = () => {
  estimateGenerated.value = true;
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.estimate-tool {
  margin: 0 auto;
  text-align: center;
}

.product {
  margin: 10px 0;
}

button {
  margin-top: 20px;
  padding: 10px;
  background-color: #4caf50;
  color: white;
  border: none;
  cursor: pointer;
}

button:hover {
  background-color: #45a049;
}

.total {
  margin-top: 20px;
}

h1 {
  color: #333;
}

h2 {
  color: #444;
}

h3 {
  margin-top: 30px;
}

h4 {
  color: #333;
}

ul {
  list-style-type: none;
  padding: 0;
}
</style>
