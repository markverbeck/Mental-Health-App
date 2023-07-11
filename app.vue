<template>
  <div class="bg-slate-200 h-screen">
    <div v-if="store.authenticated" class="w-full bg-purple-500 flex justify-between items-center p-3 border-b-2 border-black">
    <div class="flex">
      <h3 @click="hoverBox" class="text-white flex items-center ">Hello <span @mouseover="hoverBox"   class="font-bold pl-1"> {{store.user.displayName}}</span>! <div :class="store.user?.status" class="ml-1 mb-2 h-3 w-3 rounded-full border-black border-2" ></div></h3>
      <NuxtLink to="/dashboard" class="text-white ml-2">
        <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
          <path stroke-linecap="round" stroke-linejoin="round" d="M3 12l2-2m0 0l7-7 7 7M5 10v10a1 1 0 001 1h3m10-11l2 2m-2-2v10a1 1 0 01-1 1h-3m-6 0a1 1 0 001-1v-4a1 1 0 011-1h2a1 1 0 011 1v4a1 1 0 001 1m-6 0h6" />
        </svg>
      </NuxtLink>
      <NuxtLink to="/profile" class="text-white ml-2 relative">
        <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
          <path stroke-linecap="round" stroke-linejoin="round" d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z" />
        </svg>
        <div v-if="store.user.contactRequests.length > 0" class="h-3 w-3 rounded-full border-black border-2 bg-red-500 absolute right-0" style="top: -6px;"></div>
      </NuxtLink>
    </div>
      <div  @mouseleave="hoverBox" class="absolute p-3 bg-white top-12 w-1/6" v-if="state.hoverBoxOpen">
        <p> <span class="font-bold">ID:</span> {{store.user.uid}}</p>
      </div>
      <h3 class="text-white font-bold">{{store.getLocation}}</h3>
      <button class="rounded p-3 bg-pink-400 text-white border-2 border-red-600" @click="logOut">Log Out</button>
    </div>
    <NuxtLayout>
      <NuxtPage/>
    </NuxtLayout>
    <div class="w-full absolute bottom-0 bg-purple-500 p-3 flex justify-evenly" v-if="store.authenticated">
      <button @click="updateStatus('bg-green-500')" class="flex justify-center items-center content-center bg-gradient-to-br from-green-300 to-green-600 shadow-md hover:shadow-lg h-20 w-20 rounded-full fill-current text-white">
        <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
          <path stroke-linecap="round" stroke-linejoin="round" d="M14.828 14.828a4 4 0 01-5.656 0M9 10h.01M15 10h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
        </svg>
      </button>

      <button @click="updateStatus('bg-yellow-500')" class="flex justify-center items-center content-center bg-gradient-to-br from-yellow-300 to-yellow-600 shadow-md hover:shadow-lg h-20 w-20 rounded-full fill-current text-white">
        <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
          <path stroke-linecap="round" stroke-linejoin="round" d="M9.172 16.172a4 4 0 015.656 0M9 10h.01M15 10h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
        </svg>
      </button>

      <button @click="updateStatus('bg-red-500')" class="flex justify-center items-center content-center bg-gradient-to-br from-red-300 to-red-600 shadow-md hover:shadow-lg h-20 w-20 rounded-full fill-current text-white">
        <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
          <path stroke-linecap="round" stroke-linejoin="round" d="M12 8v4m0 4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
        </svg>
      </button>
    </div>
  </div>
</template>

<script setup>
import { userStore } from './store/user'
import { reactive } from 'vue'
import { initializeApp } from 'firebase/app';
import { getAuth, onAuthStateChanged } from "firebase/auth";
import { getFirestore, getDoc, doc, updateDoc, onSnapshot } from "firebase/firestore";

const state = reactive({ 
        hoverBoxOpen: false,
        auth: null,
        db: null
    })

const store = userStore()
const route = useRoute()

onBeforeMount(async () => {
  
   const app = initializeApp(store.firebaseConfig);
   const auth = getAuth(app);
   store.db = getFirestore(app)
   store.auth = auth;

   onAuthStateChanged(auth, async (user) => {
    if (user) {
      const theUser = await onSnapshot(doc(store.db, 'users', user.uid), (doc) => {
        store.setUser(doc.data())
        if(!store.authenticated){
          store.authenticate()
        }
        if(route.fullPath === '/'){
        return navigateTo('/dashboard')
        }
      })
       
      
    } else {
        store.un_authenticate()
        // store.setUser(null)
      if(route.fullPath !== '/'){
        return navigateTo('/')
      }
    }
});
})

const logOut = () => {
  store.auth.signOut()
}

const updateStatus = async (status) => {
  store.updateStatus(status)
  await updateDoc(doc(store.db, 'users', store.user.uid), {
    status: status
  });
}

const hoverBox = () => {
 state.hoverBoxOpen = !state.hoverBoxOpen
}


</script>



