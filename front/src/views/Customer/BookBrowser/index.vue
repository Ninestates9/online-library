<template>
  <div class="rightmain">
    <div class="righttop">
      <div class="mt-4">
        <el-input v-model="searchInput" placeholder="请输入搜索内容" class="input-with-select" size="large">
          <template #prepend>
            <el-select v-model="store.searchOP" placeholder="类型" style="width: 115px" size="large">
              <el-option label="书号" value=" Bno" />
              <el-option label="书名" value="Bname" />
              <el-option label="出版社" value="press" />
              <el-option label="关键字" value="key" />
              <el-option label="第一作者" value="author1" />
              <el-option label="第二作者" value="author2" />
              <el-option label="第三作者" value="author3" />
              <el-option label="第四作者" value="author4" />
            </el-select>
          </template>
          <template #append>
            <el-button :icon="Search" @click="getBookBrowser" />
          </template>
        </el-input>
      </div>
    </div>
    <div class="bookbrowser">
      <vanta-birds />
      <div v-if="isShow" class="slide" >
        <el-carousel class="slide-content">
          <el-carousel-item :key="1" class="slide-content-item">
            <img :src="slide1" alt="slide1" class="slide-img" style="object-fit: fill;"/>
          </el-carousel-item>
          <el-carousel-item :key="2" class="slide-content-item">
            <img :src="slide2" alt="slide2" class="slide-img" style="object-fit: cover;"/>
          </el-carousel-item>
          <el-carousel-item :key="3" class="slide-content-item">
            <img :src="slide3" alt="slide3" class="slide-img" style="object-fit: cover;"/>
          </el-carousel-item>
          <el-carousel-item :key="4" class="slide-content-item">
            <img :src="slide4" alt="slide4" class="slide-img" style="object-fit: cover;"/>
          </el-carousel-item>
        </el-carousel>
      </div>
      <el-card @click="openPurchaseModal(book)" v-for="book in books" :key="book.Bno" class="book-card">
        <div class="book-mini-card">
          <img :src="store.ip + '/cover/' + book.cover" alt="书本封面" class="book-cover" />
          <div class="book-info">
            <div class="book-info-left">
              <div class="book-info-bookname">{{ book.Bname }}</div>
              <div>{{ book.authors.join(' ') }}</div>
              <div>{{ book.keys.join(' ') }}</div>
              <div> {{ book.press }}</div>
            </div>
            <div class="book-info-right">
              <strong class="price">¥{{ book.price }}</strong>
            </div>
          </div>
        </div>
      </el-card>
      <el-dialog v-model="isModalOpen" title="书籍详情">
        <el-descriptions :border="true" column="3">
          <el-descriptions-item :rowspan="5" :width="200" label="" label-width="0px" align="center">
            <img :src="store.ip + '/cover/' + selectedBook?.cover" alt="书本封面" class="book-cover" />
          </el-descriptions-item>

          <el-descriptions-item label="书号"> {{ selectedBook?.Bno }}</el-descriptions-item>
          <el-descriptions-item label="丛书号">{{ selectedBook?.Bsubno }}</el-descriptions-item>
          <el-descriptions-item label="书名">{{ selectedBook?.Bname }}</el-descriptions-item>
          <el-descriptions-item label="作者">{{ selectedBook?.authors.join(' ') }}</el-descriptions-item>
          <el-descriptions-item label="关键字">{{ selectedBook?.keys.join(' ') }}</el-descriptions-item>
          <el-descriptions-item label="出版社">{{ selectedBook?.press }}</el-descriptions-item>
          <el-descriptions-item label="库存">{{ selectedBook?.quantity }}</el-descriptions-item>
          <el-descriptions-item label="位置"> {{ selectedBook?.position }}</el-descriptions-item>
          <el-descriptions-item label="简介">{{ selectedBook?.catalog }}</el-descriptions-item>


        </el-descriptions>
        <template #footer>
          <strong class="price" style="margin-left: 15%;"> ¥{{ selectedBook?.price }}</strong>
          <el-input-number v-model="orderQuantity" :min="1" label="选择数量" style="margin-right: 20px;" />
          <el-button @click="isModalOpen = false">取消</el-button>
          <el-button type="primary" @click="addToCart">确认购买</el-button>
        </template>
      </el-dialog>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { ref, onMounted } from 'vue';
import { mainStore } from '../../../store/index.ts';
import { ElCard, ElRow, ElCol, ElMessage } from 'element-plus';
import axios from 'axios';
import { Search } from '@element-plus/icons-vue'
import VantaBirds from '../../VantaBirds.vue';
import slide1 from "../../../assets/images/slide1.jpg"
import slide2 from "../../../assets/images/slide2.jpg"
import slide3 from "../../../assets/images/slide3.jpg"
import slide4 from "../../../assets/images/slide4.png"
// 初始化 Pinia store
const store = mainStore();
const searchInput = ref(''); // 搜索输入框绑定的值
const isModalOpen = ref(false);
const selectedBook = ref(null);
const orderQuantity = ref(1);
const isShow = ref(false);
const searchOptions = [
  { value: 'Bno', label: '书号' },
  { value: 'Bname', label: '书名' },
  { value: 'press', label: '出版社' },
  { value: 'key', label: '关键字' },
  { value: 'author1', label: '第一作者' },
  { value: 'author2', label: '第二作者' },
  { value: 'author3', label: '第三作者' },
  { value: 'author4', label: '第四作者' }
];

const selectedOption = ref('Bno'); // 默认选中的值


const openPurchaseModal = (book) => {
  selectedBook.value = {
    Bno: book.Bno,
    Bsubno: book.Bsubno,
    Bname: book.Bname,
    authors: book.authors,
    press: book.press,
    price: book.price,
    quantity: book.quantity,
    cover: book.cover, // Include the cover image for detail view
    keys: book.keys,
    catalog: book.catalog,
    position: book.position,
  };
  orderQuantity.value = 1; // Reset quantity
  isModalOpen.value = true;
};

// 更新选中的搜索选项
const updateSearchOption = () => {
  store.searchOP = selectedOption.value; // 保存到 store
  // alert(store.searchOP);
};

// 书籍数据
const books = ref([

]);


const addToCart = () => {
  if (!selectedBook.value) return;

  const cartItem = {
    Bno: selectedBook.value.Bno,   // 书号
    Bsubno: selectedBook.value.Bsubno, // 书号子编号
    Bname: selectedBook.value.Bname, // 书名
    authors: selectedBook.value.authors, // 作者
    press: selectedBook.value.press, // 出版社
    price: selectedBook.value.price, // 单价
    orderNumber: orderQuantity.value, // 购买数量
    total: selectedBook.value.price * orderQuantity.value, // 总价（你自己计算）
  };
  console.log(selectedBook.value.Bno);
  store.cartItems.push(cartItem); // 添加到购物车
  ElMessage.success('已成功添加到购物车！');
  isModalOpen.value = false; // 关闭弹窗
};


const getBookBrowser = () => {
  // alert(store.searchOP);
  let formData = new FormData();
  formData.append('type', store.searchOP);
  formData.append('content', searchInput.value);
  axios({
    method: 'post',
    data: formData,
    url: `${store.ip}/api/searchBooks`,
    headers: { 'Content-Type': 'multipart/form-data' }
  })
    .then(response => {
      let responseData = response.data;
      console.log(responseData);
      if (responseData.ret === 1) {
        ElMessage({ message: responseData.msg, type: 'error', duration: 5 * 1000, grouping: true });
      } else {
        isShow.value = false;
        books.value = responseData.books;
      }
    }).catch(error => {
      console.error('Error fetching books:', error);
      ElMessage.error('请求失败，请稍后重试');
    });
}

onMounted(() => {
  isShow.value = true;
});
</script>

<style scoped>
.rightmain {
  position: relative;
  display: grid;
  height: 98.5vh;
  width: 100%;
  grid-template-rows: 15% 85%;
}

.righttop {
  position: relative;
  overflow: hidden;
  width: 100%;
  height: 100%;
}

.input-with-select {
  max-width: 600px;
  margin-top: 5%;
  background-color: #ffffff7b;
}

:deep(.el-input) {
  padding-bottom: 10px;
  background-color: #59f9ff00;
}

.search-input {
  width: 80%;
  padding: 10px;
  margin-left: 3%;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.searchbtn {
  background-color: rgba(163, 43, 218, 0.5);
  color: white;
  border: 1.5px solid rgba(176, 185, 15, 0.5);
  max-height: 100%;
  margin-right: 5px;
}

.search-options {
  display: flex;
  flex-wrap: wrap;
  /* 允许换行 */
  margin-top: 10px;
  /* 添加顶部边距 */
}

.search-options label {
  margin-right: 10px;
  /* 添加右侧间距 */
  color: #333;
  /* 可根据需要调整颜色 */
}

.bookbrowser {
  display: flex;
  /* 使用 flexbox 进行布局 */
  flex-wrap: wrap;
  /* 允许换行 */
  justify-content: flex-start;
  /* 左对齐 */
  width: 100%;
  /* 确保容器宽度为 100% */
  padding: 10px;
  overflow-y: auto;
  /* 允许垂直滚动 */
}

.book-card {
  flex: 0 1 calc(23% - 20px);
  /* 每个卡片占据三分之一的宽度，减去边距 */
  display: flex;
  flex-direction: column;
  /* 垂直排列内容 */
  justify-content: space-between;
  /* 内容均匀分布 */
  height: 500px;
  /* 固定高度 */
  margin: 10px;
  /* 卡片之间的间距 */
  border-radius: 8px;
  /* 圆角 */
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  /* 阴影效果 */
  box-sizing: border-box;
  /* 包含边距和填充在宽高内 */
  background-color: #ffffff7b;
  cursor: pointer;
}

.book-cover {
  width: 100%;
  height: 300px;
  object-fit: cover;
}

.book-mini-card {
  display: flex;
  flex-direction: column;

}

.book-info {
  margin-top: 5%;
  margin-bottom: 5%;
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  align-items: center;
}

.book-info-left {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
}

.book-info-bookname {
  font-size: 25px;
}

.price {
  display: flex;
  /* 使用 flexbox 进行布局 */
  justify-content: flex-start;
  position: relative;
  font-size: 30pt;
  color: red;

}

.slide {
  width: 100%;
  height: 80%;
}

.detail-book-cover {
  width: 50%;
  height: auto;
  margin-bottom: 10px;
}

.slide-content {
  height: 100%;
}

:deep(.el-carousel__container) {
  height: 100%;
  position: relative;
}

.slide-content-item{
  height: 100%;
  width: 100%;
}

.slide-img {
  width: 100%;
  height: 100%;
}

</style>