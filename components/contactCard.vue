<template>
    <div class="w-full flex flex-row bg-neutral-100 p-3 mb-3 border-2 border-purple-500 shadow relative" :class="state.checkIn || state.response? 'animate-pulse' : '' " v-if="state.contact" @mouseover="state.checkingIn = true" @mouseleave="state.checkingIn = false">
        <button v-if="!state.modelOpen" class="absolute w-8 h-8 rounded-full bg-pink-400 border-2 border-red-600 drop-shadow text-white" style="top:-18px; right: -14px;" @click="state.modelOpen = true">x</button>
                <div class="w-2/12">
                    <div  class="flex justify-center items-center content-center shadow-md hover:shadow-lg h-12 w-12 rounded-full fill-current text-white" :class="state.contact.status">
                        <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
                            <path stroke-linecap="round" stroke-linejoin="round" d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z" />
                        </svg>
                    </div>
                </div>
                <div class="w-5/12 flex items-center pt">
                    <div>
                        <p>{{state.contact.displayName}}</p>
                    </div>
                </div>
                <div class="w-5/12 flex items-center pt justify-evenly ease-in duration-100" :class="state.checkingIn? 'opacity-100' : 'opacity-0' ">
                    <div @click="checkIn('Call')" class="hover:cursor-pointer hover:scale-125 ease-in duration-100" v-if="!state.checkIn && !state.response && !state.checkedIn">
                        <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
                            <path stroke-linecap="round" stroke-linejoin="round" d="M3 5a2 2 0 012-2h3.28a1 1 0 01.948.684l1.498 4.493a1 1 0 01-.502 1.21l-2.257 1.13a11.042 11.042 0 005.516 5.516l1.13-2.257a1 1 0 011.21-.502l4.493 1.498a1 1 0 01.684.949V19a2 2 0 01-2 2h-1C9.716 21 3 14.284 3 6V5z" />
                        </svg>
                    </div>
                    <div @click="checkIn('In person')" class="hover:cursor-pointer hover:scale-125 ease-in duration-100" v-if="!state.checkIn && !state.response && !state.checkedIn">
                        <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
                            <path stroke-linecap="round" stroke-linejoin="round" d="M12 4.354a4 4 0 110 5.292M15 21H3v-1a6 6 0 0112 0v1zm0 0h6v-1a6 6 0 00-9-5.197M13 7a4 4 0 11-8 0 4 4 0 018 0z" />
                        </svg>
                    </div>
                    <p v-if="state.checkIn" >{{state.userCheckIn?.method}}</p>
                    <p v-if="state.response" @click="acknowledge()" class="hover:cursor-pointer">{{state.userResponse.response}}</p>
                    <div class="hover:cursor-pointer hover:scale-125 ease-in duration-100" v-if="state.checkIn" @click="respond('yes')">
                        <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
                            <path stroke-linecap="round" stroke-linejoin="round" d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z" />
                        </svg>
                    </div>
                    <div @click="respond('no')" class="hover:cursor-pointer hover:scale-125 ease-in duration-100" v-if="state.checkIn">
                        <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
                            <path stroke-linecap="round" stroke-linejoin="round" d="M10 14l2-2m0 0l2-2m-2 2l-2-2m2 2l2 2m7-2a9 9 0 11-18 0 9 9 0 0118 0z" />
                        </svg>
                    </div>
                </div>
                <div v-if="state.modelOpen" class="absolute inset-0 w-full bg-pink-400 border-2 border-red-600 drop-shadow text-white text-center p-2">
                    <h2>Remove {{state.contact.displayName}} from your contact list?</h2>
                    <div>
                        <button  @click="removeContact" class="mr-3">Yes</button>
                        <button @click="state.modelOpen = false">Cancel</button>
                    </div>
                </div>
            </div>
</template>

<script setup>
    import { userStore } from '../store/user'
    import { onSnapshot, doc, setDoc } from "firebase/firestore";
    const store = userStore()
    const props = defineProps({
    id: String
    })

    onMounted(() => {
        const contact = onSnapshot(doc(store.db, "users", props.id), (doc) => {
            state.contact = doc.data()
            if(state.contact.status === 'bg-red-500' && !state.userAlerted ){
                alert(`${state.contact.displayName}'s status has gone red.  They are currently at ${state.contact.location} and need assistance`)
                state.userAlerted = true;
            } else if (state.contact.status !== 'bg-red-500') {
                state.userAlerted = false;
            }

            state.contact.checkIn.forEach(check => {
                if(check.user === store.user.uid){
                    state.userCheckIn = check
                    state.userResponse = null
                }
            })

            store.user.checkIn.forEach(check => {
                if(check.user === state.contact.uid){
                   state.checkedIn = true;
                }
            })
            
            state.contact.response.forEach(res => {
                if(res.user === store.user.uid){
                    state.userCheckIn = null
                    state.userResponse = res
                }
            })

            
            state.checkIn = state.userCheckIn? true : false
            state.response = state.userResponse? true : false
        });
    })

    const state = reactive({ 
        auth: null,
        db: null,
        contact: null,
        checkingIn: false,
        checkIn: false,
        response: false,
        userCheckIn: null,
        userResponse: null,
        checkedIn: false,
        userAlerted: false,
        modalOpen: false
    })

    const checkIn = async (method) => {
        const checkIn =  {
                user: state.contact.uid,
                method: method
            }
        const user = store.user
        user.checkIn.push(checkIn)
        state.checkedIn = true;
        await setDoc(doc(store.db, "users", store.user.uid), user)
    }

    const respond = async (res) => {
        state.checkIn = false
        state.userCheckIn = null
        state.checkedIn = false;
        const response = {
                user: state.contact.uid,
                response: `${store.user.displayName}'s response is ${res}`
        }
        store.user.response.push(response)
        state.contact.checkIn.forEach((check, index) => {
            if(check.uid === store.user.id){
                state.contact.checkIn.splice(index, 1)
            }
        })
        await setDoc(doc(store.db, "users", store.user.uid), store.user)
        await setDoc(doc(store.db, "users", state.contact.uid), state.contact)
        
    }

    const acknowledge = async () => {
        state.checkedIn = false;
        state.response = false;
        state.userResponse = null
        state.contact.response.forEach((res, index) => {
                if(res.user === store.user.uid){
                    state.contact.response.splice(index, 1)
                }
            })
        await setDoc(doc(store.db, "users", state.contact.uid), state.contact)
    }

   const removeContact = async () => {
       store.removeContact(state.contact)
       state.contact.contacts.forEach((cont, index) => {
           if(cont.uid === store.user.uid){
               state.contact.contacts.splice(index, 1)
           }
       })

       await setDoc(doc(store.db, "users", store.user.uid), store.user)
       await setDoc(doc(store.db, "users", state.contact.uid), state.contact)
       state.modalOpen = false;
   }
   
</script>
