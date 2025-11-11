<template>
  <div
    class="min-h-screen flex items-center justify-center bg-cover bg-center bg-no-repeat"
    style="background-image: url('https://digital-sign.soechi.com/Content/Images/bg.jpg')"
  >
    <div
      class="w-full max-w-md bg-white/95 rounded-2xl shadow-xl p-8 text-center backdrop-blur-sm"
    >
      <!-- Logo -->
      <div class="flex justify-center mb-6">
        <img
          src="https://digital-sign.soechi.com/Content/Images/logo%20only.png"
          alt="Soechi Logo"
          class="h-16 w-auto"
        />
      </div>

      <!-- Form -->
      <form class="mt-6 space-y-5" @submit.prevent="makeLoginRequest">
        <div>
          <input
            v-model="email"
            type="text"
            name="username"
            placeholder="User ID"
            class="w-full rounded-lg border border-blue-300 px-4 py-2.5 bg-white focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-blue-500 transition"
            required
          />
        </div>
        <div>
          <input
            v-model="password"
            type="password"
            name="password"
            placeholder="Password"
            class="w-full rounded-lg border border-blue-300 px-4 py-2.5 bg-white focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-blue-500 transition"
            required
          />
        </div>

        <div class="text-left">
          <a href="/forgot-password" class="text-sm text-blue-600 hover:underline">
            Forgot password?
          </a>
        </div>

        <button
          type="submit"
          :disabled="loggingIn"
          class="w-full bg-blue-600 text-white font-medium rounded-lg px-4 py-2.5 hover:bg-blue-700 transition disabled:opacity-50 disabled:cursor-not-allowed"
        >
          <span v-if="!loggingIn">Login</span>
          <span v-else class="animate-pulse">Logging in...</span>
        </button>
      </form>
    </div>
  </div>
</template>

<script setup>
import { onMounted, ref } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import session from '../session'
import LoginBox from './LoginBox.vue'

const loggingIn = ref(null)
const email = ref(null)
const password = ref(null)
const errorMessage = ref(null)
const redirectRoute = ref(null)

const route = useRoute()
const router = useRouter()
onMounted(() => {
	if (route?.query?.route) {
		redirectRoute.value = route.query.route
		router.replace({ query: null })
	}
})
const makeLoginRequest = async () => {
	if (!email.value || !password.value) {
		return
	}
	try {
		errorMessage.value = null
		loggingIn.value = true
		let res = await session.login(email.value, password.value)
		if (res) {
			router.push(redirectRoute.value || '/')
		}
	} catch (error) {
		console.error(error)
		errorMessage.value = error.messages.join('\n')
	} finally {
		loggingIn.value = false
	}
}
</script>
