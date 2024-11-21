<template>
  <div class="estimate-tool">
    <h1 class="mb-10 text-4xl font-bold">託送・発電管理システム見積</h1>
    <div class="fpinvoice-editor">
      <div class="fpinvoice">
        <div>Plan</div>
        <div>Opt</div>
        <div>明細項目名</div>
        <div>単価</div>
        <div>上書単価</div>
        <div>数量</div>
        <div>小計</div>
        <div class="billing-items">
          <div class="billing-items-first"></div>
          <div class="billing-items-second"></div>
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
          <input
            type="number"
            placeholder=""
            class="text-right planPrice"
            readonly
          />
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
            class="calculateCost"
            readonly
          />
        </div>
        <div
          v-for="(user, index) in users"
          :key="'users-' + index"
          class="billing-items"
        >
          <div class="billing-items-first"></div>
          <div class="billing-items-second"></div>
          <input
            type="text"
            :value="user.name"
            placeholder="明細項目名"
            class="pl-10 item-name"
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
            :value="calculateCost(user)"
            placeholder="0"
            class="calculateCost"
            readonly
          />
        </div>
        <div
          v-for="(stCost, index) in startUpCost"
          :key="'startup-' + index"
          class="billing-items"
        >
          <div class="billing-items-first"></div>
          <div class="billing-items-second"></div>
          <input
            type="text"
            :value="stCost.name"
            placeholder="明細項目名"
            class="pl-10 item-name"
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
            class="calculateCost"
            readonly
          />
        </div>
        <div
          v-for="(product, index) in products"
          :key="index"
          class="billing-items"
        >
          <div
            class="billing-items-first"
            v-if="product.type !== 'product'"
          ></div>
          <div class="checkbox-option" v-if="product.type !== 'productIp'">
            <input
              :id="'product-option-' + index"
              :disabled="product.selected_nomal || isDisabledOption(product)"
              type="checkbox"
              v-model="product.selected_option"
              @change="handleCheckboxChange(product)"
            />
          </div>
          <div class="checkbox-nomal">
            <input
              :id="'product-nomals-' + index"
              :disabled="product.selected_option"
              type="checkbox"
              v-model="product.selected_nomal"
              @change="handleCheckboxChange(product)"
            />
          </div>
          <label
            placeholder="項目名"
            class="product-name item-name"
            :for="'product-' + index"
            >{{ product.name }}</label
          >
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
            class="text-right product-quantity"
            readonly
          />
          <input
            type="number"
            :value="calculateCost(product)"
            placeholder="0"
            class="calculateCost"
            readonly
          />
        </div>
        <div
          v-for="(storage, index) in product_storage"
          :key="'storage-' + index"
          class="billing-items"
        >
          <div class="billing-items-first"></div>
          <div class="billing-items-second"></div>
          <input
            type="text"
            :value="storage.name"
            placeholder="明細項目名"
            class="pl-10 item-name"
            readonly
          />
          <input
            type="number"
            :value="storage.price"
            placeholder="単価"
            class="text-right"
            readonly
          />
          <input
            type="number"
            v-model.number="storage.variablePrice"
            placeholder="単価_変動時用"
            class="text-right"
          />
          <input
            type="number"
            v-model.number="storage.quantity"
            placeholder="数量"
            class="text-right"
          />
          <input
            type="number"
            :value="calculateCost(storage)"
            placeholder="0"
            class="calculateCost"
            readonly
          />
        </div>
      </div>
    </div>
    <div class="current-total">
      <div class="total">
        <p>小計金額 : {{ formatNumber(totalPriceExcludingTax) }} 円</p>
      </div>
      <div class="total">
        <p>消費税(10％) : {{ formatNumber(totalPriceTax) }} 円</p>
      </div>
      <div class="total">
        <p>合計金額 : {{ formatNumber(totalPrice) }} 円</p>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch } from "vue";

//プラン情報
const plans = ref([
  { name: "ベーシック プラン", basePrice: 32000, optionPrice: 0 },
  { name: "スタンダード プラン", basePrice: 40000, optionPrice: 15000 },
  { name: "コンプリート プラン", basePrice: 50000, optionPrice: 30000 },
]);

//初期導入費用
const startUpCost = ref([
  {
    type: "startup",
    name: "システム初期導入費用",
    price: 100000,
    quantity: 1,
    variablePrice: 0,
  },
]);

const users = ref([
  {
    type: "user",
    name: "ユーザーアカウント ※追加分として3アカウント毎追加料金 : 5,000 円",
    price: 0,
    quantity: 1,
    variablePrice: 0,
  },
]);

// オプションサービス
const products = ref([
  {
    type: "productIp",
    name: "接続元IPアドレスによるアクセス制限機能",
    price: 1000,
    selected_nomal: false,
    quantity: 0,
    variablePrice: 0,
  },

  {
    type: "product",
    name: "パスワード自動解除機能",
    price: 5000,
    selected_option: false,
    selected_nomal: false,
    quantity: 0,
    variablePrice: 0,
  },
  {
    type: "product",
    name: "確定値、速報値自動取得機能",
    price: 5000,
    selected_option: false,
    selected_nomal: false,
    quantity: 0,
    variablePrice: 0,
  },
  {
    type: "product",
    name: "30分速報値自動取得機能",
    price: 25000,
    selected_option: false,
    selected_nomal: false,
    quantity: 0,
    variablePrice: 0,
  },
  {
    type: "product",
    name: "発電関連帳票自動取得機能",
    price: 5000,
    selected_option: false,
    selected_nomal: false,
    quantity: 0,
    variablePrice: 0,
  },
  {
    type: "product",
    name: "再エネ卸帳票自動取得機能",
    price: 10000,
    selected_option: false,
    selected_nomal: false,
    quantity: 0,
    variablePrice: 0,
  },
  {
    type: "product",
    name: "沖縄関連帳票自動取得機能",
    price: 5000,
    selected_option: false,
    selected_nomal: false,
    quantity: 0,
    variablePrice: 0,
  },
  {
    type: "product",
    name: "全銀データ自動作成機能",
    price: 5000,
    selected_option: false,
    selected_nomal: false,
    quantity: 0,
    variablePrice: 0,
  },
]);

const product_storage = ref([
  {
    type: "storage",
    name: "各請求書等の長期保存機能 / 年",
    price: 0,
    quantity: 1,
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

//オプションのチェックボックスを押すと数量がの値を1にする
function handleCheckboxChange(product) {
  if (product.type === "productIp") {
    const user = users.value.find((user) => user.type === "user");
    if (user) {
      if (product.selected_nomal) {
        product.quantity = user.quantity;
      } else {
        product.quantity = 0;
      }
    }
  } else if (product.selected_nomal) {
    product.quantity = 1;
  } else if (product.selected_option) {
    product.quantity = 1;
  } else {
    product.quantity = 0;
  }
}

const totalPlanPackOptPrice = computed(() => {
  return products.value
    .filter((product) => product.selected_option)
    .reduce((sum, product) => sum + product.price, 0);
})

function isDisabledOption(product) {
  if(product.selected_option){
    return false
  }
  if (!selectedPlan.value || selectedPlan.value.optionPrice === undefined)
    return true;
  const remainingPrice = selectedPlan.value.optionPrice - totalPlanPackOptPrice.value;
  return remainingPrice < product.price;
}


watch(
  () => users.value.map((user) => user.quantity), // users 배열의 quantity만 감지
  () => {
    // 유저의 quantity가 변경될 때마다 해당 "productIp" 타입 제품의 quantity 업데이트
    products.value.forEach((product) => {
      if (product.type === "productIp" && product.selected_nomal) {
        const user = users.value.find((user) => user.type === "user");
        if (user) {
          product.quantity = user.quantity;
        }
      }
    });
  },
  { deep: true } // 깊은 watch로 내부 프로퍼티 변화도 감지
);

function handleUserAndStoragebox(item) {
  if (item.type === "storage") {
    item.price = item.quantity < 2 ? 0 : 5000;
  } else if (item.type === "user") {
    item.price = item.quantity < 2 ? 0 : 2000;
  }
}

watch(
  () => [
    ...users.value.map((item) => item.quantity),
    ...product_storage.value.map((item) => item.quantity),
  ],
  () => {
    users.value.forEach((item) => {
      handleUserAndStoragebox(item);
    });
    product_storage.value.forEach((item) => {
      handleUserAndStoragebox(item);
    });
  },
  { deep: true }
);

//各項目のtotalCostを計算
function calculateCost(item) {
  const unitPrice = item.variablePrice > 0 ? item.variablePrice : item.price;
  const baseCost = unitPrice * item.quantity;
  const usercount = item.quantity;
  if (item.type === "user") {
    if (usercount > 1) {
      const firstUserPrice = 2000;
      const discountMultiplier = Math.floor((usercount - 1) / 3); //３の倍数ごとに割引
      const discount = discountMultiplier * 1000;
      return baseCost - firstUserPrice - discount;
    }
  } else if (
    (item.type === "storage" && usercount > 1) ||
    item.selected_option
  ) {
    const discount = item.variablePrice > 0 ? item.variablePrice : item.price;
    return baseCost - discount;
  }
  return baseCost;
}

// 総合額計算
const totalPriceExcludingTax = computed(() => {
  const usersAccountTotal = users.value.reduce(
    (sum, user) => sum + calculateCost(user),
    0
  );
  const startupTotal = startUpCost.value.reduce(
    (sum, stCost) => sum + calculateCost(stCost),
    0
  );
  const optionsTotal = products.value.reduce(
    (sum, product) =>
      product.selected_nomal ? sum + calculateCost(product) : sum,
    0
  );
  const product_storageTotal = product_storage.value.reduce(
    (sum, storage) => sum + calculateCost(storage),
    0
  );
  return (
    monthlyFee.value +
    usersAccountTotal +
    startupTotal +
    optionsTotal +
    product_storageTotal
  );
});

const totalPriceTax = computed(() => {
  const taxRate = 0.1; // 税率
  return Math.ceil(totalPriceExcludingTax.value * taxRate);
});

const totalPrice = computed(() => {
  return totalPriceExcludingTax.value + totalPriceTax.value;
});

const formatNumber = (number) => {
  return new Intl.NumberFormat("en-US").format(number);
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.estimate-tool {
  width: 1200px;
  margin: 0 auto;
  text-align: center;
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
  margin-top: 5px;
  margin-left: 40px;
  font-weight: bold;
  font-size: 20px;
}

h3 {
  margin-top: 30px;
}

h4 {
  color: #333;
}

.fpinvoice {
  max-width: 1200px;
  margin: auto;
  display: grid;
  grid-template-columns: 3% 3% 44% 11% 11% 10% 14%;
  margin-bottom: 2rem;
  gap: 0.5rem;
}

.billing-items {
  display: contents;
}

.billing-items-first,
.billing-items-second {
  border: 1px solid gray;
  border-radius: 5px;
  background-color: #f5f5f5;
}

.product-name {
  border: 1px solid gray;
  border-radius: 5px;
  text-align: left;
  padding-left: 5px;
  padding-top: 4px;
}

.checkbox-nomal,
.checkbox-option {
  border: 1px solid gray;
  border-radius: 5px;
  padding-top: 5px;
  accent-color: #0075ff;
}

input[type="text"],
input[type="number"] {
  border: 1px solid gray;
  padding: 3px;
  border-radius: 5px;
}

input[placeholder="単価"] {
  background-color: #f5f5f5;
}

.planPrice {
  background-color: #f5f5f5;
}

.calculateCost {
  background-color: #f5f5f5;
  text-align: right;
}

.product-quantity,
.item-name {
  background-color: #f5f5f5;
}

select {
  border: 1px solid gray;
  padding: 5px;
  border-radius: 5px;
}

.current-total {
  display: flex;
  justify-self: right;
}
</style>
