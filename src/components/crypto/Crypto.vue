<template>
  <div>
    <password :passwordData="raw" v-if="passwordPanelVisible"></password>
    <v-textarea
        v-model="raw"
        color="teal"
        v-show="!hideRawText"
    >
      <template v-slot:label>
        <div>原始数据</div>
      </template>
    </v-textarea>

    <v-radio-group
        v-model="selectedCryptoType"
        row
        mandatory
    >
      <v-radio
          v-for="item in cryptoType"
          :label="item"
          :value="item"
          :key="item"
      ></v-radio>
    </v-radio-group>

    <v-text-field
        prepend-icon="mdi-key-chain-variant"
        v-model="key"
        type="password"
        v-if="selectedCryptoType && cryptoObj[selectedCryptoType].key"
        v-show="!hidePassword"
    >
    </v-text-field>

    <v-btn
        class="ma-1"
        @click="encode"
    >
      加密
    </v-btn>
    <v-btn
        class="ma-1"
        @click="decode"
    >
      解密
    </v-btn>

    <v-textarea
        v-model="decodeStr"
        color="teal"
    >
      <template v-slot:label>
        <div>加密数据</div>
      </template>
    </v-textarea>
  </div>
</template>

<script>
import CryptoJS from 'crypto-js'
import cryptoObj from './methods'
import Password from "@/components/Password";

export default {
  name: "Crypto",
  components: {Password},
  data: () => ({
    raw: "",
    decodeStr: "",
    selectedCryptoType: null,
    key: "",
    cryptoType: [
        "AES",
    ],
    passwordPanelVisible: false,
    count: 0,
    cryptoObj,
    hidePassword: false,
    hideRawText: false,
    inputsHidden: false,
    inCooldown: false,
    countResetTimer: null,
    cooldownTimer: null,
  }),
  created(){
    window.c = CryptoJS
  },
  beforeDestroy() {
    if (this.countResetTimer) clearTimeout(this.countResetTimer)
    if (this.cooldownTimer) clearTimeout(this.cooldownTimer)
  },
  watch: {
    count(){
      if (this.inCooldown || this.count <= 10) return
      this.toggleInputsHidden()
      this.startCooldown()
    }
  },
  methods:{
    showPasswordAction(){
      if (this.inCooldown) return
      if (this.count === 0) {
        this.countResetTimer = setTimeout(() => {
          this.count = 0
          this.countResetTimer = null
        }, 5000)
      }
      this.count = this.count + 1
    },
    toggleInputsHidden() {
      if (this.countResetTimer) {
        clearTimeout(this.countResetTimer)
        this.countResetTimer = null
      }
      this.count = 0
      if (this.inputsHidden) {
        this.showInputs()
      } else {
        this.hideInputs()
      }
    },
    hideInputs() {
      this.inputsHidden = true
      this.passwordPanelVisible = true
      this.decodeStr = localStorage.getItem('decodeStr')
      this.hidePassword = true
      this.hideRawText = true
    },
    showInputs() {
      this.inputsHidden = false
      this.hidePassword = false
      this.hideRawText = false
    },
    startCooldown() {
      this.inCooldown = true
      this.cooldownTimer = setTimeout(() => {
        this.inCooldown = false
        this.cooldownTimer = null
      }, 5000)
    },
    decode(){
      this.showPasswordAction()
      this.raw = cryptoObj[this.selectedCryptoType].decode(this.decodeStr, this.key)
    },
    encode(){
      this.decodeStr = cryptoObj[this.selectedCryptoType].encode(this.raw, this.key)
      localStorage.setItem('decodeStr', this.decodeStr)
    }
  }
}
</script>

<style scoped>

</style>
