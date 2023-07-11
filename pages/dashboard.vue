<template>
<div class="flex items-center h-4/6">
    <div class="w-3/4 flex flex-row m-auto pt-16 h-full ">
        <div class="w-1/3 border-r-2 border-stone-400 text-center pr-5">
            <h3 class="font-bold underline mb-5">Find friend by ID</h3>
            <form @submit.prevent="search" class="flex mb-8">
                <input v-model="state.id" type="text" class="w-1/2 p-2 rounded-l-lg drop-shadow">
                <button class="w-1/2 p-2 rounded-r-lg bg-pink-400 text-white border-2 border-red-600 drop-shadow">Search</button>
            </form>
            <p v-if="state.searhError" class="text-red-500">{{state.searhError}}</p>
            <div v-if="state.searchResults" class="w-full flex flex-row bg-neutral-100 p-3 mb-3 border-2 border-purple-500 shadow rounded ">
                <div class="w-2/3 flex items-center">
                    <p>{{state.searchResults.displayName}}</p>
                </div>
                <div class="w-1/3 flex items-center justify-end">
                    <button class="w-8 h-8 rounded-full bg-pink-400 border-2 border-red-600 drop-shadow text-white" @click="contactRequest">+</button>
                </div>
                
            </div>

            <h3 class="font-bold underline mb-5 mt-8">Add/change location</h3>
            <form @submit.prevent="" class="flex mb-8">
                <input v-model="state.location" type="text" class="w-1/2 p-2 rounded-l-lg drop-shadow">
                <button class="w-1/2 p-2 rounded-r-lg bg-pink-400 text-white border-2 border-red-600 drop-shadow flex items-center justify-center" @click="updateLocation">Add</button>
            </form>
        </div>
        
        <div class="w-2/3 px-8 overflow-auto pt-5" v-if="store.user">
            <contactCard v-for="contact in store.user.contacts" :id="contact.uid" />
        </div>
    </div>
</div>
    
</template>

<script setup>
    import { userStore } from '../store/user'
    import { reactive } from 'vue'
    import { initializeApp } from 'firebase/app';
    import { getAuth, onAuthStateChanged } from "firebase/auth";
    import { getFirestore, getDoc, doc, updateDoc, setDoc } from "firebase/firestore";
    import contactCard from '../components/contactCard'
    const store = userStore()

    const state = reactive({ 
        id: '',
        location: '',
        private: false ,
        auth: null,
        db: null,
        searchResults: null,
        searhError: null
    })

    const search = async () => {
        const theUser = await getDoc(doc(store.db, 'users', state.id))
       if(theUser.exists()){
           state.searchResults = theUser.data()
       }else{
           state.searhError = "Sorry, the user you requested could not be found."
       }
    }

    const contactRequest = async () => {
        state.id = ''
        state.searchResults.contactRequests.push(store.user)
        const docRef = await setDoc(doc(store.db, "users", state.searchResults.uid), state.searchResults)
        state.searchResults = null   
    }

    const updateLocation = async () => {
        store.updateLocation(state.location)
        await updateDoc(doc(store.db, 'users', store.user.uid), {
            location: state.location
        });
        state.location = ''
    }

    const locationPrivacy = () => {
        store.updateLocationPrivacy()
    }

</script>

<style lang="scss" scoped>

</style>