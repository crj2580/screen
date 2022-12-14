<script setup lang="ts">
import { onMounted, ref } from 'vue';
import axios from "axios"

interface InformationInterface {
  id: number,
  brandId: number,
  itemImagesOrVideos: {
    id: number,
    fileUrl: string
  }[],
  itemId: string,
  title: string,
  subTitle: string,
  price: number,
  description: string,
  salesVolume: number,
  skus: {
    skuId: string,
    title: string,
    weight: number
  }[]
}

const itemIndex = ref<number>(0)
const subKey = ref<string>("");
const information = ref<InformationInterface>();
const imageInfo = ref<any>();
const imageSrc = ref<string>("");
const speed = ref<number>(10)
const productDetail = ref<HTMLDivElement>()
const imageDeatil = ref<HTMLDivElement>();
const imageDeatilClone = ref<HTMLDivElement>();
const MyMar = ref<any>()
const changeImg = ref<any>()
const startY = ref<number>()
const moveY = ref<number>()

subKey.value = "ace7f3da21e54838ae83e15ca5547566";  // 设备密钥

/* 获取商品信息 */
const getProductInformaton = () => {
  axios.get(
    "https://product.api.troncell.com/api/services/app/SensingDevice/GetProducts",
    {
      params: {
        Subkey: subKey.value,
        MaxResultCount: 999,
        SkipCount: 0,
      }
    }
  ).then((response) => {
    console.log('response', response)
    if (response) {
      response.data.result.items.map((item: any) => {
        console.log('item', item)
        information.value = item
        imageInfo.value = item.itemImagesOrVideos[0].fileUrl
      })
    }
    console.log('information.value', information!.value!.itemImagesOrVideos);
    imageSrc.value = information!.value!.description.replace("<p>", "").replace("</p>", "");
    console.log('imageSrc.value', imageSrc.value)
  })
    .catch((error) => {
      console.log('error', error)
    })
}

/* 点击切换缩略图 */
const handleChange = (index: number, img: any) => {
  itemIndex.value = index
  imageInfo.value = img
  clearInterval(changeImg.value)
  thumbnailRotation()
}

/* 长图滚动 */
const MarQuee = () => {
  if (imageDeatilClone.value?.offsetHeight) {
    if (imageDeatilClone.value!.offsetHeight - productDetail.value!.scrollTop < 4) {
      productDetail.value!.scrollTop -= imageDeatil.value!.offsetHeight
    } else {
      productDetail.value!.scrollTop++
    }
  }
}

/* 手指触摸开始 */
const touchstart = (e: any) => {
  e.preventDefault();
  startY.value = e.touches[0].clientY
  clearInterval(MyMar.value)
}

/* 手指触摸结束 */
const touchend = (e: any) => {
  e.preventDefault()
  MyMar.value = setInterval(MarQuee, speed.value);
}

/* 手指触摸移动 */
const touchmove = (e: any) => {
  moveY.value = e.touches[0].clientY - startY.value!
  productDetail.value!.scrollTop -= moveY.value / 6
}

/* 缩略图自动切换 */
const thumbnailRotation = () => {
  changeImg.value = setInterval(() => {
    itemIndex.value++
    if (itemIndex.value == 5) {
      itemIndex.value = 0
      imageInfo.value = information.value?.itemImagesOrVideos[0].fileUrl
    }
    information.value?.itemImagesOrVideos.map((item) => {
      if (item.id == information.value?.itemImagesOrVideos[itemIndex.value].id) {
        imageInfo.value = item.fileUrl
      }
    })
  }, 2000)
}

onMounted(() => {
  getProductInformaton();
  MyMar.value = setInterval(MarQuee, speed.value);
  thumbnailRotation()
})


</script>

<template>
  <div class="screen w-100 pos-re">
    <!-- 背景图 -->
    <img src="./assets/img/bg.png" class="bg" alt="">
    <!-- 左侧大图 -->
    <img :src="imageInfo" class="img1 pos-ab" alt="">
    <!-- 标题 -->
    <div class="head-title flex-row pos-ab">
      <div class="title">零食</div>
      <div class="subtitle">{{information?.subTitle}}</div>
    </div>
    <!-- 价格 -->
    <div class="price flex-row pos-ab">
      <div class="price-title">RMB</div>
      <div class="price-detail">{{information?.price}}.00</div>
    </div>
    <!-- 二维码 -->
    <div class="qrcode flex-row pos-ab">
      <div class="qrcode-img pos-re">
        <img src="./assets/img/qrcode.png" class="qrimg pos-ab" alt="">
      </div>
      <div class="qrcode-title">打开微信扫码购买</div>
    </div>

    <!-- 品牌信息 -->
    <div class="detail-container ">
      <div class="detail-one flex-row pos-ab">
        <div class="brand flex-row">
          <div>品牌</div>
          <div class="detail">{{information?.title}}</div>
        </div>
        <div class="weight flex-row">
          <div>重量</div>
          <div class="detail">{{information?.skus[0].weight}}g</div>
        </div>
      </div>

      <div class="detail-two flex-row pos-ab">
        <div class="brand flex-row">
          <div>单位</div>
          <div class="detail">袋装</div>
        </div>
        <div class="expirationDate flex-row">
          <div>保质期</div>
          <div class="detail">12个月</div>
        </div>
        <div class="salesVolume flex-row">
          <div>销量</div>
          <div class="detail">{{information?.salesVolume}}件</div>
        </div>
      </div>
    </div>

    <!-- 详情图 -->
    <div class="detail-img flex-row pos-ab">
      <div v-for="(item,index) in information?.itemImagesOrVideos" @click="handleChange(index,item.fileUrl)"
        :class="['img-container',(itemIndex == index ? 'active' : '')]">
        <img :src="item.fileUrl" class="" alt="" />
      </div>
    </div>

    <!-- 商品详情 -->
    <div class="product-detail flex-col pos-ab">
      <div class="product-title">商品详情</div>

      <div ref="productDetail" @touchstart="touchstart" @touchend="touchend" @touchmove="touchmove"
        class="product-container">
        <div class="product-img">
          <div ref="imageDeatil" class="product-imgdetail" v-html="imageSrc"></div>
          <!-- <div ref="imageDeatilClone" class="product-imgdetail-clone" v-html="imageSrc"></div> -->
          <div ref="imageDeatilClone" class="demo2" v-html="imageSrc"></div>
        </div>
      </div>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.screen {
  // width: 1920px;
  height: 5.32rem;

  .bg {
    width: 100%;
    height: 100%;
  }

  .img1 {
    width: 3.64rem;
    height: 3.99rem;
    top: 0.4rem;
    left: 1.6rem;
  }

  .head-title {
    font-size: .36rem;
    top: 0.54rem;
    left: 6.45rem;
    font-family: "bold";

    .title {
      letter-spacing: 0.1rem;
      color: #fff;
    }

    .subtitle {
      margin-left: 0.5rem;
      color: #181818;
    }
  }

  .price {
    top: 1.7rem;
    left: 6.95rem;
    font-size: .6rem;
    font-family: "black";

    .price-title {
      margin-right: 0.38rem;
    }
  }

  .qrcode {
    top: 1.93rem;
    left: 10.81rem;
    width: 2.32rem;
    background-color: #fff;

    .qrcode-img {
      width: 1.7rem;
      height: .9rem;
      border: 2px solid #6e6e6e;

      .qrimg {
        top: 0.06rem;
        left: 0.05rem;
      }
    }

    .qrcode-title {
      font-size: .22rem;
      font-family: "regular";
      color: #6e6e6e;
      text-align: center;
      border: 2px solid #6e6e6e;
      border-left: none;
      padding: 0.1rem 0.22rem 0 0.22rem;
    }
  }

  .detail {
    margin-left: 0.2rem;
  }

  .detail-container {
    font-size: .24rem;
    font-family: "medium";


    .detail-one {
      left: 6.74rem;
      top: 2.79rem;
    }

    .weight {
      margin-left: 0.7rem;
    }

    .detail-two {
      left: 6.74rem;
      top: 3.23rem;

      .expirationDate {
        margin-left: 1.1rem;
      }

      .salesVolume {
        margin-left: 0.99rem;
      }
    }
  }

  .detail-img {
    top: 3.78rem;
    left: 6.38rem;

    .img-container {
      width: 1rem;
      height: 1rem;
      margin-left: 0.35rem;
    }
  }

  .product-detail {
    top: 1.6rem;
    left: 13.72rem;
    font-size: .24rem;
    font-family: "regular";

    .product-title {}

    .product-container {
      width: 4.34rem;
      height: 3.03rem;
      overflow: hidden;


      .product-img {
        height: 800%;
        background-color: #fff;


        .product-imgdetail {
          margin-bottom: .2rem;
        }
      }
    }

  }


  .active {
    border: 4px solid #ec701a;
  }
}
</style>
