<template>
  <div class="estimate-tool">
    <h1 class="mb-10 text-3xl font-bold">託送・発電管理システム見積ツール</h1>
    <div class="fpinvoice-editor">
      <div class="fpinvoice">
        <div>明細項目名</div>
        <div>単価</div>
        <div>単価 ※単価変動時</div>
        <div>数量</div>
        <div>小計</div>
        <div class="billing-items">
          <select v-model="selectedPlan" id="plan-select" @change="updatePrice">
            <option v-for="(plan, index) in plans" :key="index" :value="plan">
              {{ plan.name }}
            </option>
          </select>
          <input
            type="number"
            :value="monthlyFee"
            placeholder="単価"
            class="text-right"
            readonly
          />
          <input type="number" placeholder="" class="text-right" readonly />
          <input
            type="number"
            v-model="customerCount"
            placeholder="需要家数入力"
            class="text-right"
          />
          <input
            type="number"
            :value="monthlyFee"
            placeholder="0"
            class="text-right"
            readonly
          />
        </div>
        <div
          v-for="(user, index) in users"
          :key="'users-' + index"
          class="billing-items"
        >
          <input
            type="text"
            :value="user.name"
            placeholder="明細項目名"
            class="pl-10"
            readonly
          />
          <input
            type="number"
            :value="user.price"
            placeholder="単価"
            class="text-right"
            readonly
          />
          <input
            type="number"
            v-model.number="user.variablePrice"
            placeholder="単価_変動時用"
            class="text-right"
          />
          <input
            type="number"
            v-model.number="user.quantity"
            placeholder="数量"
            class="text-right"
          />
          <input
            type="number"
            :value="calculateCost(user, index)"
            placeholder="0"
            class="text-right"
            readonly
          />
        </div>
        <div
          v-for="(stCost, index) in startUpCost"
          :key="'startup-' + index"
          class="billing-items"
        >
          <input
            type="text"
            :value="stCost.name"
            placeholder="明細項目名"
            class="pl-10"
            readonly
          />
          <input
            type="number"
            :value="stCost.price"
            placeholder="単価"
            class="text-right"
            readonly
          />
          <input
            type="number"
            v-model.number="stCost.variablePrice"
            placeholder="単価_変動時用"
            class="text-right"
          />
          <input
            type="number"
            v-model.number="stCost.quantity"
            placeholder="数量"
            class="text-right"
          />
          <input
            type="number"
            :value="calculateCost(stCost)"
            placeholder="0"
            class="text-right"
            readonly
          />
        </div>
        <div
          v-for="(product, index) in products"
          :key="index"
          class="billing-items"
        >
          <div class="checkbox-container">
            <input
              :id="'product-' + index"
              type="checkbox"
              placeholder="項目名"
              v-model="product.selected"
              @change="handleCheckboxChange(product)"
            />
            <label class="product-name" :for="'product-' + index">{{
              product.name
            }}</label>
          </div>
          <input
            type="number"
            :value="product.price"
            placeholder="単価"
            class="text-right"
            readonly
          />
          <input
            type="number"
            v-model.number="product.variablePrice"
            placeholder="単価_変動時用"
            class="text-right"
          />
          <input
            type="number"
            placeholder="数量"
            v-model.number="product.quantity"
            class="text-right"
            readonly
          />
          <input
            type="number"
            :value="calculateCost(product)"
            placeholder="0"
            class="text-right"
            readonly
          />
        </div>
      </div>
    </div>

    <div class="total">
      <h2>見積合計金額: {{ totalPrice }} 円</h2>
    </div>
    <!-- <button @click="generateEstimate">見積詳細</button>
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
    </div> -->
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

//初期導入費用
const startUpCost = ref([
  {
    name: "システム初期導入費用",
    price: 100000,
    quantity: 1,
    variablePrice: 0,
  },
]);

const users = ref([
  {
    name: "ユーザーアカウント 追加分",
    price: 2000,
    quantity: 0,
    variablePrice: 0,
  },
]);

// 選択したプラン及び需要家数
const selectedPlan = ref(null);
const price = ref(0);
const customerCount = ref("");

function updatePrice() {
  price.value = selectedPlan.value ? selectedPlan.value.basePrice : "";
}

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
  {
    name: "パスワード自動解除機能",
    price: 5000,
    selected: false,
    quantity: 0,
    variablePrice: 0,
  },
  {
    name: "確定値、速報値自動取得機能",
    price: 5000,
    selected: false,
    quantity: 0,
    variablePrice: 0,
  },
  {
    name: "30分速報値自動取得機能",
    price: 25000,
    selected: false,
    quantity: 0,
    variablePrice: 0,
  },
  {
    name: "発電関連帳票自動取得機能",
    price: 5000,
    selected: false,
    quantity: 0,
    variablePrice: 0,
  },
  {
    name: "再エネ卸帳票自動取得機能",
    price: 10000,
    selected: false,
    quantity: 0,
    variablePrice: 0,
  },
  {
    name: "沖縄関連帳票自動取得機能",
    price: 10000,
    selected: false,
    quantity: 0,
    variablePrice: 0,
  },
  {
    name: "全銀データ自動作成機能",
    price: 5000,
    selected: false,
    quantity: 0,
    variablePrice: 0,
  },
]);

//オプションのチェックボックスを押すと数量がの値を1にする
function handleCheckboxChange(product) {
  if (product.selected) {
    product.quantity = 1;
  } else {
    product.quantity = 0;
  }
}


//各項目のtotalCostを計算
function calculateCost(item, index) {
  const unitPrice = item.variablePrice > 0 ? item.variablePrice : item.price;
  const baseCost = unitPrice * item.quantity;

  if (index < users.value.quantity) {
    const discountMultiplier = Math.floor(users.value.quantity / 3); //３の倍数ごとに割引
    const discount = discountMultiplier * 1000;
    return baseCost - discount;
  }

  return baseCost;
}

// 総合額計算
const totalPrice = computed(() => {
  const startupTotal = startUpCost.value.reduce(
    (sum, stCost) => sum + calculateCost(stCost),
    0
  );
  const optionsTotal = products.value.reduce(
    (sum, product) => (product.selected ? sum + calculateCost(product) : sum),
    0
  );
  return monthlyFee.value + startupTotal + optionsTotal;
});

// 選択されたオプション目録
// const selectedProducts = computed(() => {
//   return products.value.filter((product) => product.selected);
// });

// // 견적 생성 버튼 클릭 시 처리
// const estimateGenerated = ref(false);
// const generateEstimate = () => {
//   estimateGenerated.value = true;
// };
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

.fpinvoice {
  max-width: 1200px;
  margin: auto;
  display: grid;
  grid-template-columns: 40% 15% 15% 10% 15%;
  margin-bottom: 2rem;
  gap: 0.5rem;
}

.billing-items {
  display: contents;
}

.checkbox-container {
  display: flex;
  align-items: center;
  border: 1px solid gray;
  padding: 3px;
  border-radius: 5px;
}

input[type="checkbox"] {
  margin-right: 10px;
}

input[type="text"],
input[type="number"] {
  border: 1px solid gray;
  padding: 3px;
  border-radius: 5px;
}

select {
  border: 1px solid gray;
  padding: 5px;
  border-radius: 5px;
}
</style>
