<template>
  <div class="estimate-tool">
    <h1 class="mb-10 text-4xl font-bold">託送・発電管理システム見積</h1>
    <div class="fpinvoice-editor">
      <div class="fpinvoice">
        <div class="billing-items-menu">オプ<br />ション</div>
        <div class="billing-items-menu">プラン<br />付帯</div>
        <div class="billing-items-menu">明細項目名</div>
        <div class="billing-items-menu">単価</div>
        <div class="billing-items-menu">上書単価</div>
        <div class="billing-items-menu">数量</div>
        <div class="billing-items-menu">小計</div>
        <div
          v-for="(stCost, index) in startUpCost"
          :key="'startup-' + index"
          class="billing-items"
        >
          <div class="checkbox-nomal checkbox-wrapper-19">
            <input
              :id="'stCost-' + index"
              type="checkbox"
              v-model="stCost.selected_option"
              @change="handleCheckboxChange(stCost)"
            />
            <label :for="'stCost-' + index" class="check-box"></label>
          </div>
          <div class="billing-items-plan"></div>
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
            class="text-right product-quantity"
            readonly
          />
          <input
            type="number"
            :value="calculateCost(stCost)"
            placeholder="0"
            class="calculateCost"
            readonly
          />
        </div>
        <div class="billing-items">
          <div class="billing-items-option"></div>
          <div class="billing-items-plan"></div>
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
          <div class="billing-items-option"></div>
          <div class="billing-items-plan"></div>
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
          v-for="(product, index) in products"
          :key="index"
          class="billing-items"
          :class="{
            'hidden-print': !(
              product.selected_option || product.selected_option_plan
            ),
          }"
        >
          <div class="checkbox-nomal checkbox-wrapper-19">
            <input
              :id="'product-nomal-' + index"
              :disabled="product.selected_option_plan"
              type="checkbox"
              v-model="product.selected_option"
              @change="handleCheckboxChange(product)"
            />
            <label :for="'product-nomal-' + index" class="check-box"></label>
          </div>
          <div
            class="billing-items-plan"
            v-if="product.type !== 'product'"
          ></div>
          <div
            class="checkbox-option checkbox-wrapper-19"
            v-if="product.type !== 'productIp'"
          >
            <input
              :id="'product-option-' + index"
              :disabled="isDisabledOption(product) || product.selected_option"
              type="checkbox"
              v-model="product.selected_option_plan"
              @change="handleCheckboxChange(product)"
            />
            <label :for="'product-option-' + index" class="check-box"></label>
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
            :class="product.class"
            :readonly="product.isReadonly"
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
          <div class="billing-items-option"></div>
          <div class="billing-items-plan"></div>
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
    <div class="container-option-total">
      <div class="option-total">
        <p>プラン付帯オプション残高 : {{ formatNumber(remainingPrice) }} 円</p>
      </div>
      <div class="option-total">
        <p>ユーザーアカウント数 : {{ formatNumber(usersCount) }} 個</p>
      </div>
    </div>
    <div class="container-total">
      <div class="total">
        <span class="label">小計金額</span>
        <span class="colon">:</span>
        <span class="value">{{ formatNumber(totalPriceExcludingTax) }} 円</span>
      </div>
      <div class="total">
        <span class="label">消費税(10％)</span>
        <span class="colon">:</span>
        <span class="value">{{ formatNumber(totalPriceTax) }} 円</span>
      </div>
      <div class="total">
        <span class="label">合計金額</span>
        <span class="colon">:</span>
        <span class="value">{{ formatNumber(totalPrice) }} 円</span>
      </div>
      <button @click="myPrint" class="hidden-print">印刷</button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch } from "vue";

//初期導入費用
const startUpCost = ref([
  {
    type: "startup",
    name: "システム初期導入費用",
    price: 100000,
    selected_option: false,
    quantity: 0,
    variablePrice: 0,
  },
]);

//プラン情報
const plans = ref([
  { name: "ベーシック プラン", basePrice: 32000, optionPrice: 0 },
  { name: "スタンダード プラン", basePrice: 40000, optionPrice: 15000 },
  { name: "コンプリート プラン", basePrice: 50000, optionPrice: 30000 },
]);

const users = ref([
  {
    type: "user",
    name: "ユーザーアカウント 1アカウント毎",
    price: 0,
    quantity: 1,
    variablePrice: 0,
  },
]);

// オプションサービス
const products = ref([
  {
    type: "product",
    name: "3ユーザ利用オプション",
    price: 5000,
    selected_option: false,
    selected_option_plan: false,
    quantity: 0,
    variablePrice: 0,
    isReadonly: false,
    class: "text-right",
  },
  {
    type: "productIp",
    name: "接続元IPアドレスによるアクセス制限機能",
    price: 1000,
    selected_option: false,
    quantity: 0,
    variablePrice: 0,
    isReadonly: true,
    class: "text-right product-quantity",
  },
  {
    type: "product",
    name: "パスワード自動解除機能",
    price: 5000,
    selected_option: false,
    selected_option_plan: false,
    quantity: 0,
    variablePrice: 0,
    isReadonly: true,
    class: "text-right product-quantity",
  },
  {
    type: "product",
    name: "確定値、速報値自動取得機能",
    price: 5000,
    selected_option: false,
    selected_option_plan: false,
    quantity: 0,
    variablePrice: 0,
    isReadonly: true,
    class: "text-right product-quantity",
  },
  {
    type: "product",
    name: "30分速報値自動取得機能",
    price: 25000,
    selected_option: false,
    selected_option_plan: false,
    quantity: 0,
    variablePrice: 0,
    isReadonly: true,
    class: "text-right product-quantity",
  },
  {
    type: "product",
    name: "発電関連帳票自動取得機能",
    price: 5000,
    selected_option: false,
    selected_option_plan: false,
    quantity: 0,
    variablePrice: 0,
    isReadonly: true,
    class: "text-right product-quantity",
  },
  {
    type: "product",
    name: "再エネ卸帳票自動取得機能",
    price: 10000,
    selected_option: false,
    selected_option_plan: false,
    quantity: 0,
    variablePrice: 0,
    isReadonly: true,
    class: "text-right product-quantity",
  },
  {
    type: "product",
    name: "沖縄関連帳票自動取得機能",
    price: 5000,
    selected_option: false,
    selected_option_plan: false,
    quantity: 0,
    variablePrice: 0,
    isReadonly: true,
    class: "text-right product-quantity",
  },
  {
    type: "product",
    name: "全銀データ自動作成機能",
    price: 5000,
    selected_option: false,
    selected_option_plan: false,
    quantity: 0,
    variablePrice: 0,
    isReadonly: true,
    class: "text-right product-quantity",
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
function handleCheckboxChange(item) {
  if (item.type === "productIp") {
    if (item.selected_option) {
      item.quantity = usersCount.value;
    } else {
      item.quantity = 0;
    }
  } else if (item.selected_option) {
    item.quantity = 1;
  } else if (item.selected_option_plan) {
    item.quantity = 1;
  } else {
    item.quantity = 0;
  }
}

//プラン付帯対象選択オプションの合計
const totalPlanPackOptPrice = computed(() => {
  return products.value
    .filter((product) => product.selected_option_plan)
    .reduce((sum, product) => sum + product.price * product.quantity, 0);
});

//プラン付帯対象選択オプションの合計 - プラン付帯対象金額
const remainingPrice = computed(() => {
  if (!selectedPlan.value || selectedPlan.value.optionPrice === undefined) {
    return 0;
  }
  return selectedPlan.value.optionPrice - totalPlanPackOptPrice.value;
});

//ププラン付帯対象選択オプションの合計 - プラン付帯対象金額によるCheckbox非活性化
function isDisabledOption(product) {
  if (product.selected_option_plan) {
    return false;
  }
  if (!selectedPlan.value || selectedPlan.value.optionPrice === undefined)
    return true;
  return remainingPrice.value < product.price;
}

//ユーザーアカウント数確認関数
const usersCount = computed(() => {
  const user = users.value.find((user) => user.type === "user");
  const userOption = products.value.find(
    (userOpt) => userOpt.name === "3ユーザ利用オプション"
  );
  return user.quantity + userOption.quantity * 3;
});

//選択オプションリセットロジック
watch(
  usersCount, // usersCountの変化を監視
  (newCount) => {
    console.log(`usersCount has changed to ${newCount}`);
    // 유저의 quantity가 변경될 때마다 해당 "productIp" 타입 제품의 quantity 업데이트
    products.value.forEach((product) => {
      if (product.type === "productIp" && product.selected_option) {
        product.quantity = usersCount.value;
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
    item.selected_option_plan
  ) {
    const discount =
      item.variablePrice > 0
        ? item.variablePrice * item.quantity
        : item.price * item.quantity;
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
      product.selected_option ? sum + calculateCost(product) : sum,
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

const myPrint = window.print.bind();
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped></style>
