<template>
  <el-container class='home-container'>
    <el-header>
      <div>
        <img src="../assets/google.svg" alt="mozzlla的logo" class="home-logo">
        <span>后台管理系统</span>
      </div>
      <el-button type="info" @click="logout">退出</el-button>
    </el-header>
    <el-container>
      <el-aside :width="iscollapse==true?'64px':'200px'">
        <!-- 导航菜单区域 -->
        <div :class="iscollapse==true?'iconfont iconright':'iconfont iconleft'" @click='collapse'></div>
        <el-menu background-color="#333744" text-color="#fff"
        active-text-color="#409eff" :collapse="iscollapse" :collapse-transition='false'
        router :default-active="defaultActive" unique-opened>
          <el-submenu :index="item.id+''" v-for="item in menuList" :key="item.id">
            <template slot="title">
              <i :class="'iconfont '+iconObj[item.id]"></i>
              <span>{{item.authName}}</span>
            </template>
            <el-menu-item :index="'/'+subItem.path" v-for="subItem in item.children"
            :key="subItem.id" @click="navActive('/'+subItem.path)">
              <template slot="title">
                <i class="el-icon-menu"></i>
                <span>{{subItem.authName}}</span>
              </template>
            </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>
      <el-main>
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  data () {
    return {
      menuList: [],
      iconObj: {
        125: 'iconbussiness-man',
        103: 'iconquanxian',
        101: 'iconshangpin',
        102: 'iconding_huabanfuben',
        145: 'icondata'
      },
      iscollapse: false,
      defaultActive: ''
    }
  },
  created () {
    this.getMenuList()
    this.defaultActive = window.sessionStorage.getItem('defaultActive')
  },
  methods: {
    logout () {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    async getMenuList () {
      const { data: res } = await this.$http.get('menus')
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.menuList = res.data
      console.log(res.data)
    },
    collapse () {
      this.iscollapse = !this.iscollapse
    },
    navActive (index) {
      window.sessionStorage.setItem('defaultActive', index)
      this.defaultActive = index
    }
  }
}
</script>

<style lang="less" scoped>
  .home-container {
    height: 100%;
  }

  .el-header {
    background-color: #373d41;
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding-left: 0;
    color: #fff;
    font-size: 20px;

    >div {
      display: flex;
      align-items: center;
      >img{
        height: 50px;
      }
      >span {
        text-align: center;
      }
    }
  }

  .el-aside {
    background-color: #333744;
    .el-menu{
      border-right: 0px;
    }
  }

  .el-main {
    background-color: #eaedf1;
  }
  .iconfont{
    margin-right: 5px;
  }
  .iconleft,.iconright{
    text-align: center;
    font-size: 25px;
    background-color: #4a5064;
    cursor: pointer;
  }
</style>
