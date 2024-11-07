<template>
  <div class="estimate-tool">
    <h1>託送・発電管理システム見積ツール</h1>
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
        <li v-for="(product, index) in selectedProducts" :key="index">
          {{ product.name }} - {{ product.price }} 円
        </li>
      </ul>
      <h4>合計金額: {{ totalPrice }} 円</h4>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';

// 제품 목록 정의
const products = ref([
  { name: 'パスワード自動解除機能', price: 5000, selected: false },
  { name: '確定値、速報値自動取得機能', price: 5000, selected: false },
  { name: '30分速報値自動取得機能', price: 25000, selected: false },
  { name: '発電関連帳票自動取得機能', price: 5000, selected: false },
  { name: '再エネ卸帳票自動取得機能', price: 10000, selected: false },
  { name: '沖縄関連帳票自動取得機能', price: 10000, selected: false },
  { name: '全銀データ自動作成機能', price: 5000, selected: false },
]);

// 총 금액 계산
const totalPrice = computed(() => {
  return products.value
    .filter(product => product.selected)
    .reduce((total, product) => total + product.price, 0);
});

// 선택된 제품 목록
const selectedProducts = computed(() => {
  return products.value.filter(product => product.selected);
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
  background-color: #4CAF50;
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
</style>

