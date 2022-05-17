<template>
<LoadingApponment :active="isLoading"></LoadingApponment>
<div class="text-end">
  <button class="btn btn-primary" type="button" @click="openModal(true)">增加產品</button>
</div>
<table class="table mt-4">
  <thead>
    <tr>
      <th width="120">分類</th>
      <th>產品名稱</th>
      <th width="120">原價</th>
      <th width="120">售價</th>
      <th width="100">是否啟用</th>
      <th width="200">編輯</th>
    </tr>
  </thead>
  <tbody>
    <tr v-for="item in products" :key="item.id">
      <td>{{ item.category }}</td>
      <td>{{ item.title }}</td>
      <td class="text-right">
        {{ item.origin_price }}
      </td>
      <td class="text-right">
        {{ item.price }}
      </td>
      <td>
        <span class="text-success" v-if="item.is_enabled">啟用</span>
        <span class="text-muted" v-else>未啟用</span>
      </td>
      <td>
        <div class="btn-group">
          <button @click="openModal(false, item)" class="btn btn-outline-primary btn-sm">編輯</button>
          <button @click="openDelProductModal(item)" class="btn btn-outline-danger btn-sm">刪除</button>
        </div>
      </td>
    </tr>
  </tbody>
</table>
<productModal :product="tempProduct" ref="productModal" @update-product="updateProduct"></productModal>
<deleteModal :item="tempProduct" ref="delModal" @del-item="delProduct"></deleteModal>
<paginationComponent :pages="pagination"
    @emit-pages="getProducts" ></paginationComponent>
</template>

<script>
import paginationComponent from '../components/PaginationComponent.vue'
import productModal from '../components/ProductModal.vue'
import deleteModal from '../components/DeleteModal.vue'
export default {
  data () {
    return {
      products: [],
      pagination: {},
      tempProduct: {},
      isNew: false,
      isLoading: false
    }
  },
  methods: {
    getProducts (page = 1) {
      const api = `${process.env.VUE_APP_API}api/${process.env.VUE_APP_PATH}/admin/products/?page=${page}`
      this.isLoading = true
      this.$http.get(api)
        .then((res) => {
          this.isLoading = false
          if (res.data.success) {
            this.products = res.data.products
            this.pagination = res.data.pagination
          }
        })
    },
    openModal (isnew, item) {
      if (isnew) {
        this.tempProduct = {}
      } else {
        this.tempProduct = { ...item }
      }
      this.isNew = isnew
      const productComponent = this.$refs.productModal
      productComponent.showModal()
    },
    updateProduct (item) {
      this.tempProduct = item
      let api = `${process.env.VUE_APP_API}api/${process.env.VUE_APP_PATH}/admin/product`
      let httpMethod = 'post'
      if (!this.isNew) {
        api = `${process.env.VUE_APP_API}api/${process.env.VUE_APP_PATH}/admin/product/${item.id}`
        httpMethod = 'put'
      }
      const productComponent = this.$refs.productModal
      this.$http[httpMethod](api, { data: this.tempProduct })
        .then((res) => {
          productComponent.hideModal()
          if (res.data.success) {
            this.getProducts()
            this.emitter.emit('push-message', {
              style: 'success',
              title: '更新成功'
            })
          } else {
            this.emitter.emit('push-message', {
              style: 'danger',
              title: '更新失敗',
              content: res.data.message.join('、')
            })
          }
        })
    },
    openDelProductModal (item) {
      this.tempProduct = { ...item }
      const delComponent = this.$refs.delModal
      delComponent.showModal()
    },
    delProduct () {
      const url = `${process.env.VUE_APP_API}api/${process.env.VUE_APP_PATH}/admin/product/${this.tempProduct.id}`
      this.$http.delete(url).then((res) => {
        console.log(res.data)
        const delComponent = this.$refs.delModal
        delComponent.hideModal()
        if (res.data.success) {
          this.getProducts()
          this.emitter.emit('push-message', {
            style: 'success',
            title: '刪除成功'
          })
        } else {
          this.emitter.emit('push-message', {
            style: 'danger',
            title: '刪除失敗',
            content: res.data.message.join('、')
          })
        }
      })
    }
  },
  created () {
    this.getProducts()
  },
  components: {
    productModal,
    deleteModal,
    paginationComponent
  },
  inject: ['emitter']
}
</script>
