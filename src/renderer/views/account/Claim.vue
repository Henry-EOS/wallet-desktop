<template>
  <confirm-modal :show="true" :submitting="submitting" @confirm="submit()" @close="close">
    <div>
      <div class="row">
        <div class="row__title">交易名称</div>
        <div class="row__content">提取分红</div>
      </div>
      <div class="row">
        <div class="row__title">超级节点名称</div>
        <div class="row__content">{{bpname}}</div>
      </div>
      <div class="row">
        <div class="row__title">投票人用户</div>
        <div class="row__content">{{voter}}</div>
      </div>
      <div class="row">
        <div class="row__title">可提取金额</div>
        <div class="row__content">{{rewardAmount}}</div>
      </div>
      <div class="row">
        <div class="row__title">手续费</div>
        <div class="row__content">{{app.fee}}</div>
      </div>
      <div class="row">
        <div class="row__title">输入密码</div>
        <div class="row__content">
          <input class="input" v-model="password" type="password" placeholder="请输入投票人的钱包密码" required />
        </div>
      </div>
    </div>
  </confirm-modal>
</template>

<script>
import { mapActions, mapState } from 'vuex'

import Message from '@/components/Message'
import ConfirmModal from '@/components/ConfirmModal'
import { Actions } from '@/constants/types.constants'

export default {
  name: 'unfreeze',
  data() {
    return {
      password: '',
      submitting: false,
    }
  },
  computed: {
    voter() {
      return this.$route.params.accountName
    },
    bpname() {
      return this.$route.params.bpname
    },
    rewardAmount() {
      const bp = this.account.bpsTable && this.account.bpsTable.find(bp => this.bpname === bp.name)
      if (bp) {
        if (bp.vote) {
          return bp.vote.reward
        } else {
          return '0 EOS'
        }
      } else {
        return null
      }
    },
    ...mapState(['app', 'account']),
  },
  methods: {
    submit() {
      this.submitting = true
      this.claim({
        bpname: this.bpname,
        voter: this.voter,
        password: this.password,
      })
        .then(result => {
          Message.success('提取分红成功')
        })
        .catch(err => {
          Message.error({
            title: `${err.code ? `code: ${err.code}` : '提取分红失败'}`,
            message: err.message,
          })
          this.submitting = false
          return Promise.reject(err)
        })
        .then(result => {
          this.getAccountInfo()
          this.close()
        })
    },
    close() {
      this.$router.push({ name: 'accountDetail' })
    },
    ...mapActions({
      getAccountInfo: Actions.GET_ACCOUNT_INFO,
      claim: Actions.CLAIM,
    }),
  },
  components: {
    ConfirmModal,
  },
}
</script>

