
<template>
  <div>
    <v-app-bar
      color="primary"
      dense
      flat
      dark
    >
      <v-toolbar-title>Register</v-toolbar-title>
    </v-app-bar>
    <v-container class="pt-0 pb-0">
      <v-row>
        <v-col cols="12">          
          <div class="mt-8 text-primary text-title text-center">
           
          </div>
        </v-col>
         <v-col cols="12" class="text-center pb-0 profile-img">
        <img v-if="getLine.pictureUrl == ''" src="~/assets/profile.png" alt="" width="155">
        <img v-else :src="getLine.pictureUrl" alt="" width="155">
        </v-col>
      <v-col cols="12" class="text-center pt-2 pb-0">
        {{ getLine.displayName }}
        </v-col>
        <v-col cols="12">
          <v-form>
            <!--p class="text-center text-main mb-0 mt-4">Displayname</-->
              <v-text-field
                  v-model="form.lineid"
                  label="Line ID"
                ></v-text-field>
            <v-text-field
              v-model="form.phone"
              dense          
              :rules="phoneRules"
              @keypress="onlyNumber($event, 10)"
              label="Phone"
            ></v-text-field>
             <v-text-field
              v-model="form.email"
              type="email"
              dense      
              :rules="emailRules"
              label="Email"
            ></v-text-field>           
            <p class="text-center text-main mb-0 mt-4">Link</p>
            <v-text-field
              v-model="form.circuitid1"
              dense          
              label="Circuit Id1"
            ></v-text-field>
            <v-text-field
              v-model="form.circuitid2"
              dense          
              label="Circuit Id2"
            ></v-text-field>
            <v-btn rounded color="primary" dark class="w-100 mt-10 my-btn" @click="register">Register</v-btn>
            
            <v-btn  class="w-100 text-center my-btn outlined text-primary mt-5" @click="close">Close</v-btn>  
          </v-form>
        </v-col>
      </v-row>
    </v-container>
  </div>
</template>

<script>
const REGEX_EMAIL = /^(([^<>()[\]\\.,;:\s@"]+(\.[^<>()[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/
const REGEX_PHONE = /^[0]([0-9]{9})*$/
const REGEX_NUMBER = /^[0-9]*$/


import axios from 'axios'
import { constants } from 'buffer'

export default {
   mounted(){
    liff.init({
      liffId: '1657128604-LyreQngl'
    }).then(() => {
      if(liff.isLoggedIn()){
        liff.getProfile().then(profile => {                    
          this.$store.dispatch('setLine', profile);
         //  this.$store.dispatch('setRegister', profile.userId);
          this.isDone();
        })
      }else{
        liff.login();
      }
    })
  }, 
  computed: {
    getLine(){
      return this.$store.getters.getLine;      
    }
  }, 
  data(){
    return {
      form: {
//LINEUSERID : this.$store.getters.getLine.userId,
        lineid: this.$store.getters.getRegister.LINEID,
        phone: this.$store.getters.getRegister.PHONE,
        email: this.$store.getters.getRegister.EMAIL,
        circuitid1: this.$store.getters.getRegister.CIRCUITID1,
        circuitid2: this.$store.getters.getRegister.CIRCUITID2
      },
      modal: false,
      emailValidated: false,
      phoneValidated: false,
       emailRules: [ value => this.emailValidator(value)],
       phoneRules: [ value => this.phoneValidator(value)]
    }
  },
  methods: {
    phoneValidator(value){    
      this.phoneValidated = false  
      if(value == ''){
        return 'required'
      }
      if(REGEX_PHONE.test(value) && value.length == 10){ 
        this.phoneValidated = true
        return true
      }
      return "please input phone number"
    },
     isDone(){
      const url = `https://285d-203-155-1-3.ngrok.io/api/LiffRegister?lineuserid=${this.$store.getters.getLine.userId}`
       axios.get(url).then((res) => {
        if(res.data != null){
         // alert(res.data);
         this.$router.push('register/done');

        }
        else
        {
          alert(res.data);
        }
      }); 
     // alert('login')
    },
    emailValidator(value){
      this.emailValidated = false
      if(value == ''){
        return 'required'
      }
      if(REGEX_EMAIL.test(value)){
        this.emailValidated = true
        return true
      }
      return "email is Invalid"
    },
    onlyNumber(event, max){  
      if(event.target.value.length == 0){
        if(event.key != 0){
          return event.preventDefault()
        }
      }else{
        if(!REGEX_NUMBER.test(event.key) || event.target.value.length == max){
          return event.preventDefault()
        }
      }      
    }, 
    validate(){
      let validated = true
      const errors = []
      const validatorField = [
        'email',
        'phone',
        'circuit1'
      ]
      validatorField.forEach((field) => {
        if(this.form[field] == ''){
          validated = false
          errors.push(`${field} can not be null`)
        }
      })    
      if(!this.emailValidated){
        validated = false
        errors.push(`email is Invalid`)
      }
      if(!this.phoneValidated){
        validated = false
        errors.push(`please input phone number`)
      }
      if(!validated){
        this.$store.dispatch('setDialog', {
          isShow: true,
          title: 'Form is error',
          message: errors.map((error) => error+'<br/>').join('')
        })        
      }      
      return validated
    },  
    back() {      
      this.$router.push('/register')
    },
    register() {
      if(this.validate()){
       this.$store.dispatch('setRegister', this.form);
      //  this.$axios.post("https://localhost:5001/api/LiffRegisterApi", this.$store.getters.getRegister).then((res) => {
      //  this.$router.push('/register/done')
     // }).catch(e => console.log(e))         
    // alert(this.$store.getters.getRegister);
this.formSubmit();
      }      
    },
  formSubmit() {
  const url = 'https://285d-203-155-1-3.ngrok.io/api/LiffRegister'
     axios.post(url, {
  "LINEUSERID": this.$store.getters.getLine.userId,
  "LINEID": this.form.lineid,
  "PHONE": this.form.phone,
  "EMAIL": this.form.email,
  "CIRCUITID1":  this.form.circuitid1,
  "CIRCUITID2": this.form.circuitid2
}).then((res) => {
         this.$router.push('/register/done')
       }).catch(e => console.log(e)) 
    },
     close(){
      liff.closeWindow();
    }
  
  }
  
}



</script>

<style lang="scss" scoped>
  .v-form{
    padding: 0 10px;
  }
  .profile-img{
    img{
      border-radius: 50%;
    }
  }
  .gender-group{
    p{
      margin-bottom: 0;
      align-self: center;
      margin-right: 20px;
    }
    .circle{
      width: 45px;
      height: 45px;      
      color: #FFF;
      border-radius: 50%;
      position: relative;
      background: rgba($color: #000000, $alpha: 0.3);
      margin-right: 7px;
      &.active{
        background: #1A56BE;
      }
      svg{
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
      }
    }
  }
</style>