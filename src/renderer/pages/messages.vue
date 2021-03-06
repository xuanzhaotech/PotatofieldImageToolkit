<template>
  <el-tabs type="card" tab-position="left" id="messages">
    <el-tab-pane>
      <span slot="label" class="interactable"><i class="fas fa-envelope"></i> 消息中心</span>
      <div class="tab-content">
        <div class="container" v-if="onlyUnread && unreadMessages.length != 0">
          <div
            v-for="message in unreadMessages.slice(messageListPage * 4 - 4, messageListPage * 4)"
            :key="message.id"
            class="card-container">
            <el-card class="card interactable">
              <div>
                <div class="row">
                  <div class="subtitle">{{ message.title }}</div>
                </div>
                <div class="row">
                  <v-clamp autoresize :max-lines="3" class="text">{{ message.text }}</v-clamp>
                </div>
              </div>
              <div class="row actions">
                <div class="action active" @click="showMessage(message)">
                  <span class="fa fa-envelope-open"></span>
                  <div>阅读消息</div>
                </div>
                <div class="action">
                  <div>{{ message.pub_date }}</div>
                </div>
              </div>
            </el-card>
          </div>
        </div>
        <div class="container" v-if="!onlyUnread && messages.length != 0">
          <div
            v-for="message in messages.slice(messageListPage * 4 - 4, messageListPage * 4)"
            :key="message.id"
            class="card-container">
            <el-card class="card interactable">
              <div>
                <div class="row">
                  <div class="subtitle">{{ message.title }}</div>
                </div>
                <div class="row">
                  <v-clamp autoresize :max-lines="3" class="text">{{ message.text }}</v-clamp>
                </div>
              </div>
              <div class="row actions">
                <div class="action active" @click="showMessage(message)">
                  <span class="fa fa-envelope-open"></span>
                  <div>阅读消息</div>
                </div>
                <div class="action">
                  <div>{{ message.pub_date }}</div>
                </div>
              </div>
            </el-card>
          </div>
        </div>
        <div class="container" v-if="(onlyUnread && unreadMessages.length == 0) || (!onlyUnread && messages.length == 0)">
          <div  class="empty-container">
            <div class="empty">
              <i class="fas fa-envelope-open"></i>
              <div>没有未读消息</div>
            </div>
          </div>
        </div>
        <div class="row">
          <el-pagination
            v-if="onlyUnread"
            class="interactable"
            small
            background
            layout="prev, pager, next"
            :pager-count="5"
            :page-size="4"
            :total="unreadMessages.length"
            :current-page="messageListPage"
            @current-change="messageListPageChange">
          </el-pagination>
          <el-pagination
            v-else
            class="interactable"
            small
            background
            layout="prev, pager, next"
            :pager-count="5"
            :page-size="4"
            :total="messages.length"
            :current-page="messageListPage"
            @current-change="messageListPageChange">
          </el-pagination>
          <el-switch
            v-model="onlyUnread"
            active-text="仅显示未读消息"
            inactive-text="显示全部消息"
            @change="switchMessage"
            class="interactable"></el-switch>
        </div>
      </div>
    </el-tab-pane>
    <el-tab-pane disabled>
      <span slot="label" id="sidebar">
        <div id="tool-info">
          <i id="tool-logo" class="fas fa-envelope"></i>
          <div class="text">消息中心</div>
        </div>
        <div id="control-button-holder">
          <div class="control-button interactable" @click="minimize">
            <i class="fas fa-angle-double-down"></i>
            <div>最小化</div>
          </div>
          <div class="control-button interactable" @click="close">
            <span class="fas fa-sign-out-alt"></span>
            <div>退出</div>
          </div>
        </div>
      </span>
    </el-tab-pane>
  </el-tabs>
</template>

<script>
import { ipcRenderer } from 'electron'

export default {
  name: 'messages',
  data () {
    return {
      messages: [],
      messageListPage: 1,
      onlyUnread: true
    }
  },
  computed: {
    unreadMessages() {
      return this.messages.filter((message) => {
        return this.$store.state.messages.messageList.indexOf(message.id) == -1
      })
    }
  },
  methods: {
    minimize() {
      ipcRenderer.send('minimize')
    },
    close() {
      ipcRenderer.send('close')
      this.$destroy()
    },
    messageListPageChange(page) {
      this.messageListPage = page
    },
    switchMessage() {
      this.messageListPage = 1
    },
    showMessage(message) {
      this.$dialog({
        title: message.title,
        text: message.text
      })
      if (this.$store.state.messages.messageList.indexOf(message.id) == -1) {
        this.$store.dispatch('messages/markRead', message.id)
      }
    }
  },
  mounted() {
    this.$dialog({
      title: '正在获取消息',
      text: '正在向服务器请求消息列表，请稍候。',
      showConfirm: false
    }).then((dialog) => {
      this.$http.get('https://imagetoolkit.potatofield.cn/messages').catch((error) => {
        dialog.change({
          type: 'error',
          title: '出现错误',
          text: '获取消息列表失败，请检查您的网络。',
          showConfirm: true,
          confirmFunction: () => {
            this.close()
          }
        })
      }).then((res) => {
        this.messages = res.data.map((message) => {
          let list = message.pub_date.split('-')
          let pub_date = list[0] + ' 年 ' + list[1] + ' 月 ' + list[2] + ' 日'
          message.pub_date = pub_date
          return message
        })
        dialog.close()
      })
    })
  }
}
</script>

<style lang="scss">
#messages {
  width: 100%;
  height: 100%;
  
  button {
    font-family: var(--main-font);
  }
  
  .el-tabs__header {
    margin-right: 0;
    
    .el-tabs__nav-scroll {
      background-color: var(--dark-gray);
      
      .el-tabs__nav {
        border: 0;
        height: 100%;
        display: flex;
        flex-direction: column;
        
        .el-tabs__item {
          width: 150px;
          height: 50px;
          line-height: 50px;
          color: var(--light-gray);
          text-align: center;
          border: 0;
          transition: 0.2s;
          
          #sidebar {
            width: 100%;
            
            @keyframes shine {
              0% {
                color: var(--light-gray)
              }
              25% {
                color: var(--light-gray)
              }
              50% {
                color: var(--main-color)
              }
              75% {
                color: var(--light-gray)
              }
              100% {
                color: var(--light-gray)
              }
            }
            
            #tool-info {
              display: flex;
              flex-direction: column;
              align-items: center;
              animation: shine 5s infinite;
              
              #tool-logo {
                font-size: 60px;
                margin: 20px;
              }
            }
            
            #control-button-holder {
              width: 100%;
              display: flex;
              justify-content: space-between;
              align-items: center;
              padding: 20px;
              box-sizing: border-box;
              
              .control-button {
                font-size: 12px;
                line-height: initial;
                cursor: pointer;
                transition: 0.2s;
                
                svg {
                  font-size: 20px;
                  margin: 5px;
                }
                
                &:hover {
                  color: var(--white);
                }
                
                &:active {
                  filter: brightness(0.9);
                }
              }
            }
          }
          
          &.is-active {
            background-color: var(--white);
            color: var(--main-color);
            cursor: default;
          }
          
          &.is-disabled {
            flex-grow: 1;
            padding: 0;
            display: flex;
            align-items: flex-end;
          }
          
          &:hover:not(.is-disabled):not(.is-active) {
            color: var(--white);
          }
          
          &:active:not(.is-disabled):not(.is-active) {
            filter: brightness(0.9);
          }
        }
      }
    }
  }
  
  .el-tabs__content {
    height: 100%;
    
    .el-tab-pane {
      width: 100%;
      height: 100%;
      
      .tab-content {
        width: 100%;
        height: 100%;
        padding: 20px;
        box-sizing: border-box;
        display: flex;
        flex-direction: column;
        justify-content: space-between;
      }
    }
  }
  
  .row {
    width: 100%;
    flex-shrink: 0;
    margin-top: 10px;
    margin-bottom: 10px;
    display: flex;
    flex-direction: row;
    justify-content: space-between;
    align-items: center;
    
    .el-switch {
      display: flex;
      justify-content: flex-end;
    }
    
    &:first-child {
      margin-top: 0;
    }
    
    &:last-child {
      margin-bottom: 0;
    }
  }
  
  .bar-button {
    width: 0;
    height: 28px;
    flex-grow: 1;
    box-sizing: border-box;
    border: none;
    padding-left: 0;
    padding-right: 0;
    margin-left: 5px;
    margin-right: 5px;
    
    &:first-child {
      margin-left: 0;
    }
    
    &:last-child {
      margin-right: 0;
    }
  }
  
  .container {
    width: 100%;
    height: 0;
    flex-grow: 1;
    display: flex;
    flex-wrap: wrap;
    
    .card-container {
      width: 50%;
      height: 50%;
      box-sizing: border-box;
      padding: 10px;
      
      .card {
        width: 100%;
        height: 100%;
        color: var(--dark-gray);
        
        .el-card__body {
          width: 100%;
          height: 100%;
          box-sizing: border-box;
          display: flex;
          flex-direction: column;
          
          .subtitle {
            white-space: nowrap;
            overflow: hidden;
            text-overflow: ellipsis;
          }

          .text {
            text-indent: 2em;
          }
          
          .actions {
            width: 100%;
            flex-grow: 1;
            align-items: flex-end;
            
            .action {
              display: flex;
              align-items: center;
              color: var(--gray);
              font-size: 12px;
              transition: 0.2s;
              
              svg {
                font-size: 14px;
                margin-right: 5px;
              }
            }
            
            .active {
              color: var(--dark-gray);
              cursor: pointer;
              
              &:hover {
                color: var(--main-color);
              }
              
              &:active {
                filter: brightness(0.9);
              }
            }
          }
        }
        
        &:hover {
          transform: scale(1.05);
        }
      }
    }
  }
  
  .empty-container {
    width: 100%;
    flex-grow: 1;
    display: flex;
    justify-content: center;
    align-items: center;
    
    .empty {
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
      font-size: 14px;
      
      svg {
        font-size: 40px;
        margin: 14px;
      }
    }
  }
  
  .el-pagination {
    padding: 0;
    margin-right: 10px;
    
    li {
      min-width: 24px;
      height: 28px;
      line-height: 28px;
    }
    
    .btn-prev {
      width: 24px;
      height: 28px;
      line-height: 28px;
      margin-left: 0;
    }
    
    .btn-next {
      width: 24px;
      height: 28px;
      line-height: 28px;
      margin-right: 0;
    }
  }
}
</style>
