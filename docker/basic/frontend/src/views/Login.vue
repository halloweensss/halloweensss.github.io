<template>
    <div>
        <vue-headful title="Вход"/>
    <div class="login-container d-flex align-items-center justify-content-center">
        <form class="login-form" method="send" @submit.prevent="submitHandler">
            <img class="icon-header" src="../assets/img/icon.svg">
            <div class="form-group">
                <input id="email"
                       type="text"
                       v-model.trim="email"
                       :class="{invalid: ($v.email.$dirty && !$v.email.required) || ($v.email.$dirty && !$v.email.email) || this.result ==='userNotCreated'}"
                       class="form-control input"
                       placeholder="Электронный адрес">
                <small id = "emailEmpty"
                        class="helper-text invalid"
                        v-if="$v.email.$dirty && !$v.email.required"
                >Поле не должно быть пустым</small>
                <small id = "emailIncorrect"
                        class="helper-text invalid"
                        v-else-if="$v.email.$dirty && !$v.email.email"
                >Введите корректный электронный адрес</small>
                <small id = "userNotFound"
                        class="helper-text invalid"
                        v-else-if="this.result ==='userNotCreated'"
                >Пользователь не найден</small>
            </div>
            <div class="form-group">
                <input id="password"
                       type="password"
                       v-model.trim="password"
                       :class="{invalid: ($v.password.$dirty && !$v.password.required) || this.result ==='passwordIncorrect'}"
                       class="form-control input"
                       placeholder="Пароль">
                <small
                        id = "passwordEmpty"
                        class="helper-text invalid"
                        v-if="$v.password.$dirty && !$v.password.required"
                >Введите пароль</small>
                <small
                        id = "passwordIncorrect"
                        class="helper-text invalid"
                        v-else-if="this.result ==='passwordIncorrect'"
                >Неверный пароль</small>
            </div>
            <div class="form-group">
                <input id = "submit" type="submit" class="form-control input submit login" value="Войти">
            </div>
            <div class="strike">
                <span> или </span>
            </div>
            <router-link to="/register" :style="{opacity: .9}">
                <a id = "toRegister" class="form-control input submit create-account-btn">Создать аккаунт</a>
            </router-link>
            <router-link to="/restore">
                <a id = "toRestore" class="forgot-pass">Забыли пароль?</a>
            </router-link>
        </form>
    </div>
    </div>
</template>

<script>
    import {email, required} from 'vuelidate/lib/validators'
    import HTTP from "../components/http";

    export default {
        name: "Login",
        data: () => ({
            email: '',
            password: '',
            accessToken: '',
            result: '',
            error:''
        }),
        validations: {
            email: {email, required},
            password : {required}
        },
        created(){
            this.accessToken = this.getCookie('accessToken');
            this.getProfileInfoOpen();
        },
        methods:{
            getCookie(name){
                var results = document.cookie.match ( '(^|;) ?' + name + '=([^;]*)(;|$)' );
                if ( results )
                    return ( unescape ( results[2] ) );
                else
                    return null;
            },
            saveToken(token){
                this.accessToken = token;
                document.cookie = "accessToken="+this.accessToken;
            },
            getProfileInfoOpen() {
                HTTP.post('/user/get-profile', {
                    accessToken: this.getCookie('accessToken')
                }).then(
                    (response) => {
                        this.result = response.data.status;
                        console.log(this.result);
                        if(this.result === "success"){
                            this.$router.push('/');
                        }
                    }
                )
            },
            sendLogin(){
                HTTP.post('/user/login', {
                    email: this.email,
                    password: this.password,
                    accessToken: this.accessToken
                }).then(
                    (response) => {
                        if(response.data.status === "tokenExists"){
                            this.saveToken(response.data.accessToken);
                            this.$router.push('/');
                        }
                        this.result = response.data.status;
                        if(this.result === 'success'){
                            this.saveToken(response.data.accessToken);
                            this.$router.push('/');
                        }

                    },
                    (error) =>{
                        this.result = error.response.data;
                        console.log(error.response.data.result);
                    }
                )
            },
            submitHandler(){
                if(this.$v.$invalid){
                    this.$v.$touch();
                    return;
                }else{
                    this.sendLogin();
                }
            }
        }
    }
</script>

<style scoped>
    html{
        font-size: 16px;
    }

    hr{
        color: #000000;
    }
    body{
        background-color: #EDF0FA;
        font-family: 'Roboto', sans-serif;
    }

    a{
        color:#000000;
        text-decoration: none;
        opacity: 0.5;
        text-align: center;
        display: block;
        margin: auto;
    }

    span{
        opacity: .6;
    }

    a:hover,
    a:focus{
        color: #6C71F8;
        text-decoration: none;
    }

    .helper-text.invalid{
        color: rgba(243,33,89,.8);
    }

    .forgot-pass{
        font-size: 12px;
    }
    .icon-header{
        width: 40px;
        height: 40px;
        max-width: 40px;
        max-height: 40px;
        display: block;
        margin: auto auto 20px;
    }

    .login-container{
        height: 100vh;
        width: 100%;
    }

    .login-form{
        max-width: 100%;
        width: 340px;
        padding: 15px;
        margin: auto;
    }
    .form-group{
        margin-bottom: 8px;
    }

    .create-account-btn{
        width: 50%;
        opacity: .9;
        display: block;
        margin: auto;
    }
    .input{
        font-size: 15px;
        border-radius: 5px;
        border: none;
        opacity: 1;
    }

    .input.invalid{
        border: 1px solid rgba(243,33,89,.5);
    }

    .input.invalid:focus{
        box-shadow: 0 0 2px 1px rgba(243,33,89,.5);
        border: 1px solid rgba(243,33,89,.1);
    }

    .input:focus{
        box-shadow: 0 0 1px 2px rgba(108,113,248,.5);
    }

    .input::placeholder{
        color: #000000;
        opacity: .4;
    }

    .input.submit{
        background-color: #2ADE3F;
        color: #FFFFFF;
    }

    .input.submit.login{
        background-color: #6C71F8;
        color: #FFFFFF;
    }


    .strike {
        display: block;
        text-align: center;
        overflow: hidden;
        white-space: nowrap;
    }

    .strike > span {
        position: relative;
        display: inline-block;
    }

    .strike > span:before,
    .strike > span:after {
        content: "";
        position: absolute;
        top: 50%;
        width: 9999px;
        height: 1px;
        background: black;
        opacity: 0.2;
    }

    .strike > span:before {
        right: 100%;
        margin-right: 15px;
    }

    .strike > span:after {
        left: 100%;
        margin-left: 15px;
    }
    .avatar-input:hover .avatar-input-hint,
    .avatar-input:focus .avatar-input-hint{
        visibility: visible;
    }
</style>