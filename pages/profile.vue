<template>
    <div class="w-full flex flex-row p-12 h-4/6">
        <div class="w-1/2 border-r-2 border-stone-400 h-full">
            <h2 class="font-bold mb-5">Profile</h2>
            <p><strong>name:</strong> {{store.user.displayName}}</p>
            <p><strong>Email:</strong> {{store.user.email}}</p>
            <p><strong>ID:</strong> {{store.user.uid}}</p>
        </div>
        <div class="w-1/2 pl-8 pt-8">
            <h2 class="font-bold mb-5">Contact Requests</h2>
            <div v-for="request in store.user.contactRequests" class="w-full flex flex-row bg-neutral-100 p-3 mb-3 border-2 border-purple-500 shadow rounded ">
                <div class="w-2/3 flex items-center">
                    <p>{{request.displayName}}</p>
                </div>
                <div class="w-1/3 flex items-center justify-end">
                    <button class="w-8 h-8 rounded-full bg-pink-400 border-2 border-red-600 drop-shadow text-white flex justify-center items-center mr-2" @click="addFriend(request)">
                        <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
                            <path stroke-linecap="round" stroke-linejoin="round" d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z" />
                        </svg>
                    </button>
                    <button class="w-8 h-8 rounded-full bg-pink-400 border-2 border-red-600 drop-shadow text-white flex justify-center items-center" @click="declineRequest(request)">
                        <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
                            <path stroke-linecap="round" stroke-linejoin="round" d="M10 14l2-2m0 0l2-2m-2 2l-2-2m2 2l2 2m7-2a9 9 0 11-18 0 9 9 0 0118 0z" />
                        </svg>
                    </button>
                </div>
            </div>
        </div>
    </div>
</template>

<script setup>
    import { userStore } from '../store/user'
    import { setDoc, doc, getDoc} from "firebase/firestore";
    const store = userStore()

    const addFriend = async (request) => {
        const user = await getDoc(doc(store.db, 'users', request.uid))
        const data = user.data()
        data.contacts.push(store.user)
        store.addFriend(request)
        store.removeContactRequest(request)
        const docRef = await setDoc(doc(store.db, "users", store.user.uid), store.user)
        const docRef_ = await setDoc(doc(store.db, "users", data.uid), data)
    }

    const declineRequest = async (request) => {
        store.removeContactRequest(request)
        const docRef = await setDoc(doc(store.db, "users", store.user.uid), store.user)
    }
</script>

<style lang="scss" scoped>

</style>