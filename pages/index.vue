<template>
    <div>
        <div v-if="!state.signUp" class="min-h-screen bg-slate-200 py-6 flex flex-col justify-center relative overflow-hidden sm:py-12">
            <span class="border text-4xl text-yellow-800 px-6 pt-10 pb-8 bg-white w-1/2 max-w-md mx-auto rounded-t-md sm:px-10">Sign in Form</span>
        <div class="border relative px-4 pt-7 pb-8 bg-white shadow-xl w-1/2 max-w-md mx-auto sm:px-10 rounded-b-md">
                <form @submit.prevent="signIn">
                    <label for="" class="block">Username or Email</label>
                    <input v-model="state.signInInput.email" type="Email" class="border w-full h-10 px-3 mb-5 rounded-md" placeholder="Username or Email">
                    <label for="" class="block">Password</label>
                    <input v-model="state.signInInput.password" type="password" class="border w-full h-10 px-3 mb-5 rounded-md" placeholder="password">
                            <div class="flex items-start">
                                <div class="flex items-start">
                                    <div class="flex items-center">
                                        <input id="remember" aria-describedby="remember" type="checkbox" class="bg-gray-50 border border-gray-300 focus:ring-3 focus:ring-blue-300 h-4 w-4 rounded dark:bg-gray-700 dark:border-gray-600 dark:focus:ring-blue-600 dark:ring-offset-gray-800" >
                                    </div>
                                        <div class="text-sm ml-3">
                                            <label for="remember" class="font-medium text-gray-900">Remember me</label>
                                        </div>
                                    </div>
                                    <a @click="state.signUp = !state.signUp" href="#" class="text-sm text-blue-700 hover:underline ml-auto dark:text-blue-500">Need to sign up?</a>
                                </div>
                <button class="mt-5 bg-green-500 hover:bg-blue-700 shadow-xl text-white uppercase text-sm font-semibold px-14 py-3 rounded">Login</button>
                </form>
        </div>
    </div>
    
    <div v-if="state.signUp" class="min-h-screen bg-slate-200 py-6 flex flex-col justify-center relative overflow-hidden sm:py-12">
            <span class="border text-4xl text-yellow-800 px-6 pt-10 pb-8 bg-white w-1/2 max-w-md mx-auto rounded-t-md sm:px-10">Sign up Form</span>
        <div class="border relative px-4 pt-7 pb-8 bg-white shadow-xl w-1/2 max-w-md mx-auto sm:px-10 rounded-b-md">
                <form @submit.prevent="signUp">
                    <label for="" class="block">Username</label>
                    <input v-model="state.signUpInput.displayName" type="text" class="border w-full h-10 px-3 mb-5 rounded-md" placeholder="Username or Email" required>
                    <label for="" class="block">Email</label>
                    <input v-model="state.signUpInput.email" type="Email" class="border w-full h-10 px-3 mb-5 rounded-md" placeholder="Username or Email" required>
                    <label for="" class="block">Password</label>
                    <input v-model="state.signUpInput.password" type="password" class="border w-full h-10 px-3 mb-5 rounded-md" placeholder="password">
                            <div class="flex items-start">
                                <div class="flex items-start">
                                    <div class="flex items-center">
                                        <input id="remember" aria-describedby="remember" type="checkbox" class="bg-gray-50 border border-gray-300 focus:ring-3 focus:ring-blue-300 h-4 w-4 rounded dark:bg-gray-700 dark:border-gray-600 dark:focus:ring-blue-600 dark:ring-offset-gray-800" >
                                    </div>
                                        <div class="text-sm ml-3">
                                            <label for="remember" class="font-medium text-gray-900">Remember me</label>
                                        </div>
                                    </div>
                                    <a @click="state.signUp = !state.signUp" href="#" class="text-sm text-blue-700 hover:underline ml-auto dark:text-blue-500">Already have an account?</a>
                                </div>
                <button class="mt-5 bg-green-500 hover:bg-blue-700 shadow-xl text-white uppercase text-sm font-semibold px-14 py-3 rounded">Login</button>
                </form>
        </div>
    </div>
    </div>
</template>

<script setup>
import { reactive, ref } from 'vue'
import { userStore } from '../store/user'
import {  createUserWithEmailAndPassword, signInWithEmailAndPassword, updateProfile } from "firebase/auth";
import {  getDoc, setDoc, doc} from "firebase/firestore";

const store = userStore()

const state = reactive({
    signInInput:{
        email: '',
        password: '',
    },
    signUpInput:{
        email: '',
        password: '',
        displayName: ''
    },
    signUp: false,
    auth: null,
    db: null
})

const signIn = async() => {
   await signInWithEmailAndPassword(store.auth, state.signInInput.email, state.signInInput.password)
    .then(async (userCredential) => {
        // Signed in 
        const user = await getDoc(doc(store.db, 'users', userCredential.user.uid))
       if(user.exists()){
           store.setUser(user.data())
            store.authenticate()
            navigateTo('/dashboard')
       }else{
           console.log('document doesnt exist')
       }
    })
    .catch((error) => {
        const errorCode = error.code;
        const errorMessage = error.message;
    });
}

const signUp = async () => {
   await createUserWithEmailAndPassword(store.auth, state.signUpInput.email, state.signUpInput.password)
    .then((userCredential) => {
         updateProfile(userCredential.user, {
                displayName: state.signUpInput.displayName
              }).then(async () => {
                try {
                  const defaultData = {
                    displayName: userCredential.user.displayName,
                    email: userCredential.user.email,
                    uid: userCredential.user.uid,
                    status: 'bg-green-500',
                    location: 'Home',
                    contacts: [],
                    groups: [],
                    checkIn: [],
                    response: [],
                    contactRequests:[]
                  }
                  const docRef = await setDoc(doc(store.db, "users", defaultData.uid), defaultData);
                  store.setUser(defaultData)
                  store.authenticate()
                  navigateTo('/dashboard')
                } catch (e) {
                  console.error("Error adding document: ", e);
                }
              })
    })
    .catch((error) => {
        const errorCode = error.code;
        const errorMessage = error.message;
    });
}

</script>

<style lang="scss" scoped>

</style>