<template>
  <div class="records">
    <div class="recordsTit">
      <span>浏览足迹</span>
      <span v-if="!edit" class="batch" @click="showEdit">批量删除</span>
      <div v-else class="saveList">
        <el-checkbox v-model="selectAll" @change="changeChecked" :true-label='1' :false-label='0'>全选</el-checkbox>
        <span class="delete" @click="deleteFun">删除</span>
        <span class="save" @click="saveList">保存</span>
      </div>
    </div>
    <div class="list sub-main" v-if="flag" v-loading="loading">
      <div class="recordsList">
        <div class="listBox">
          <div class="recordsItem" v-for="(item,index) of recordList" :key="index" :class="{selected: item.selected === 1, edit: edit}" @click="select(index)">
            <div class="mc" v-if="edit"></div>
            <div class="imgBox">
              <img :src="$api.BASEURL + item.image" alt="">
            </div>
            <div class="info">
              <h3 class="overflow">{{ item.storeName }}</h3>
              <span class="price">¥{{ item.price }}</span>
            </div>
            <div class="btnBox">
              <span @click="delFootprint(item)">删除足迹</span>
              <span @click="toProductDetail(item)">查看详情</span>
            </div>
          </div>
          <div class="clearfix"></div>
        </div>
      </div>
    </div>
    <div class="nothing sub-main" v-else>
      <img src="../../../../static/image/nothing@2x.png" alt="">
      <p class="fs20 font-color-999">你还没有浏览记录～</p>
      <router-link to="/">
          <el-button>去首页看看</el-button>
      </router-link>
    </div>
  </div>
</template>

<script>
import api from '@/api'
import {listSearchMixin} from '@/config/mixin'
export default {
  name: 'browseRecords',
  mixins: [listSearchMixin],
  data () {
    return {
      edit: false,
      recordChecked: 0,
      selectAll: 0,
      ids: [],
      times: [],
      recordList: [],
      flag: true,
      loading: false
    }
  },
  mounted () {
    this.getFootprint()
  },
  methods: {
    // 我的足迹
    getFootprint () {
      this.loading = true
      let params = {
        url: api.getAllProduct,
        method: 'GET',
        payload: {
          type: 'foot'
        }
      }
      this.sendReq(params, res => {
        if (res.status === 200) {
          let data = res.data
          let list = data.map(item => {
            item.selected = 0
            return item
          })
          this.recordList = list
          if (res.data.length) {
            this.flag = true
          } else {
            this.flag = false
          }
          this.loading = false
        } else {
          this.$message.warning(res.msg)
        }
      })
    },
    // 保存
    saveList () {
      this.edit = false
    },
    // 选择
    select (index) {
      if (this.recordList[index].selected === 1) {
        this.recordList[index].selected = 0
      } else {
        this.recordList[index].selected = 1
      }
      this.selectAll = 1
      this.recordList.map(i => {
        if (i.selected === 0) {
          this.selectAll = 0
        }
      })
    },
    // 全选
    changeChecked () {
      this.recordList.forEach(i => {
        this.$set(i, 'selected', this.selectAll)
      })
    },
    // 编辑
    showEdit () {
      if (this.recordList.length === 0) {
        return
      }
      this.edit = true
    },
    // 商品详情
    toProductDetail (item) {
      let data = {
        productId: item.pid
      }
      this.$router.push({
        path: '/productDetail',
        query: {
          proData: JSON.stringify(data)
        }
      })
    },
    // 删除浏览记录请求
    delFootprint (item) {
      let params = {
        url: api.unfavorite,
        method: 'POST',
        data: {
          id: item.pid,
          category: 'foot'
        }
      }
      this.sendReq(params, res => {
        if (res.status === 200) {
          this.$message.success('取消成功')
          this.edit = false
          this.getFootprint()
        } else {
          this.$message.warning('取消失败')
        }
      })
    },
    // 删除选中浏览记录
    deleteFun () {
      let flag = false
      let ids = []
      this.recordList.forEach(item => {
        if (item.selected === 1) {
          flag = true
          ids.push(item.pid)
        }
      })
      ids = ids.join(',')
      if (flag === false) {
        return this.$message.warning('请先选中需要删除的记录')
      }
      this.$confirm('此操作将删除选中的记录, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        let params = {
          url: api.unfavorites + ids,
          method: 'POST',
          data: {
            category: 'foot'
          }
        }
        this.sendReq(params, res => {
          if (res.status === 200) {
            this.$message.success('取消成功')
            this.edit = false
            this.getFootprint()
          } else {
            this.$message.warning('取消失败')
          }
        })
      }).catch(() => {
      })
    }
  }
}
</script>

<style lang="scss" scoped>
  .records {
    border: 1px solid #E5E5E5;
    .recordsTit {
      display: flex;
      padding: 0 25px;
      border-bottom: 1px solid #F5F5F5;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 30px;
      span {
        display: block;
        height: 60px;
        line-height: 60px;
      }
      span:first-child {
        color: $mainColor;
        font-size: 14px;
        border-bottom: 2px solid $mainColor;
      }
      .batch {
        color: #FFFFFF;
        font-size: 12px;
        width: 78px;
        text-align: center;
        height: 28px;
        line-height: 28px;
        background: $mainColor;
        cursor: pointer;
      }
      .saveList {
        display: flex;
        align-items: center;
        span {
          margin-left: 50px;
          cursor: pointer;
          color: #333333;
          font-size: 14px;
        }
        span.save {
          width: 78px;
          height: 28px;
          background: #FF7800;
          color: #FFFFFF;
          text-align: center;
          line-height: 28px;
        }
      }
    }
    .list {
      padding: 0 20px 20px 20px;
      .recordsList {
        margin-top: 20px;
        .topTime {
          color: #333333;
          font-size: 22px;
          margin-bottom: 30px;
          font-weight: bold;
        }
        .listBox {
          .recordsItem {
            width: 186px;
            float: left;
            margin:0 18px 20px 0;
            position: relative;
            .imgBox {
              width: 186px;
              height: 186px;
              border: 1px solid #E5E5E5;
              box-sizing: border-box;
              img {
                width: 100%;
                height: 186px;
              }
            }
            .info {
              border: 1px solid #E5E5E5;
              text-align: center;
              padding: 0 15px 15px;
              h3 {
                font-size: 14px;
                color: #333333;
                height: 30px;
                line-height: 30px;
                margin-top: 10px;
              }
              span {
                color: $mainColor;
                font-size: 14px;
              }
            }
            .btnBox {
              border: 1px solid #E5E5E5;
              display: flex;
              border-top: none;
              span {
                width: 50%;
                display: block;
                text-align: center;
                font-size: 14px;
                color: #333333;
                height: 30px;
                line-height: 30px;
                cursor: pointer;
              }
              span:first-child {
                border-right: 1px solid #E5E5E5;
                box-sizing: border-box;
              }
            }
          }
          .recordsItem:nth-child(5n) {
            margin-right: 0;
          }
          .edit:before {
            content: "";
            background: url("./../../../assets/images/noEdit.png");
            width: 58px;
            height: 58px;
            position: absolute;
            right: 0;
            top: 0;
            display: block;
            background-size: contain;
          }
          .selected:before {
            background: url("./../../../assets/images/edit.png");
            background-size: contain;
          }
        }
      }
      >>> .el-pagination {
        margin: 30px 0 20px 0;
      }
    }
    .sub-main{
      min-height: 400px;
    }
    .nothing{
      width: 100%;
      text-align: center;
      min-height: 400px;
      p{
          margin-bottom: 20px;
      }
      .el-button{
          background-color: #FF7800;
          color: #FFFFFF;
          font-weight: normal;
          border-radius: 0;
      }
    }
    .mc{
      position: absolute;
      width: 100%;
      height: 100%;
      opacity: 0;
    }
  }
</style>
