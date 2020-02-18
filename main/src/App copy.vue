<template>
  <div id="container">
    <header>
      <nav>
        <ol>
          <li v-for="app of apps" :key="app.name">
            <a @click="goto(app.name, app.href)">{{app.name}}</a>
          </li>
        </ol>
      </nav>
    </header>
    <div v-if="loading">loading</div>
    <!-- <div class="appContainer" v-html="content" /> -->
    <p>内容管理平台</p>
    <div class="headlesscms" style="height:300px;overflow:auto;">
      <div v-html="headlesscmsContent"></div>
    </div>
    <p>AB测试平台</p>
    <div class="decisionmanage" style="height:300px;overflow:auto;">
      <div v-html="decisionmanageContent"></div>
    </div>
  </div>
</template>

<script>
import { registerMicroApps, runAfterFirstMounted, setDefaultMountApp, start } from 'qiankun'
const genActiveRule = routerPrefix => {
  return location => location.pathname.startsWith(routerPrefix)
}
export default {
  name: 'master',
  data () {
    return {
      loading: false,
      content: null,
      headlesscmsContent: null,
      decisionmanageContent: null,
      appInfos:[
        { name: 'headlesscmsContent', entry: '//localhost:8081', href: '/headlesscmsContent', render: this.renderheadlesscmsContent },
        { name: 'decisionmanageContent', entry: '//localhost:8081', href: '/decisionmanageContent', render: this.renderdecisionmanageContent },
      ],
    }
  },
  computed:{
      apps(){
        return this.appInfos.map(app => {
          return {
            ...app,
            activeRule: genActiveRule(app.href)
          }
        })
      }

  },
  created () {
    if (!window.__POWERED_BY_QIANKUN__) {
      this.initQiankun()
    } else {
      location.reload()
    }
  },
  methods: {
    goto (title, href) {
      window.history.pushState({}, title, href)
    },
    renderCommon(contentType){
      return ({ content, loading })=> {
          this[contentType] = content
        }
    },
    renderheadlesscmsContent ({ appContent, loading }) {
      this.headlesscmsContent = appContent
      // this.loading = loading
    },
    renderdecisionmanageContent ({ appContent, loading }) {
      this.decisionmanageContent = appContent
      // this.loading = loading
    },
    render ({ appContent, loading }) {
      this.content = appContent
      this.loading = loading
    },
    initQiankun () {
      const { apps } = this
      registerMicroApps(
        apps,
        {
          beforeLoad: [
            app => {
              // eslint-disable-next-line no-console
              console.log('before load', app)
            }
          ],
          beforeMount: [
            app => {
              // eslint-disable-next-line no-console
              console.log('before mount', app)
            }
          ],
          afterUnmount: [
            app => {
              // eslint-disable-next-line no-console
              console.log('after unload', app)
            }
          ]
        }
      )
      const defaultApp = apps[0] || {}
      setDefaultMountApp(defaultApp.href)
      runAfterFirstMounted(() => {
        // eslint-disable-next-line no-console
        console.info('first app mounted')
      })
      start({ prefetch: true })
    }
  }
}
</script>

<style scoped>
a {
  color: #42b983;
  cursor: pointer;
}
.appContainer {
  margin-top: 50px;
}
</style>