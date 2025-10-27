<template>
    <div class="auth-backdrop" @click.self="$emit('close')">
        <div class="auth-card">
            <h2>{{ mode === 'login' ? 'Welcome back' : 'Create an account' }}</h2>

            <form @submit.prevent="handleSubmit">
                <div v-if="mode === 'signup'" class="form-group">
                    <label>Name</label>
                    <input v-model="name" type="text" placeholder="John Doe" required />
                </div>

                <div class="form-group">
                    <label>Email</label>
                    <input v-model="email" type="email" placeholder="you@example.com" required />
                </div>

                <div class="form-group">
                    <label>Password</label>
                    <input v-model="password" type="password" placeholder="••••••••" required />
                </div>

                <button class="primary-btn" type="submit">
                    {{ mode === 'login' ? 'Login' : 'Sign up' }}
                </button>
            </form>

            <p class="toggle">
                {{ mode === 'login' ? "Don't have an account?" : "Already have an account?" }}
                <span @click="toggleMode">
                    {{ mode === 'login' ? 'Sign up' : 'Login' }}
                </span>
            </p>
        </div>
    </div>
</template>

<script>
export default {
    data() {
        return {
            mode: 'login',
            name: '',
            email: '',
            password: ''
        }
    },
    methods: {
        toggleMode() {
            this.mode = this.mode === 'login' ? 'signup' : 'login'
        },
        handleSubmit() {
            // simulated validation
            alert(`${this.mode === 'login' ? 'Logged in' : 'Signed up'} successfully!`)
            this.$emit('close')
        }
    }
}
</script>

<style scoped>
.auth-backdrop {
    position: fixed;
    inset: 0;
    background: rgba(0, 0, 0, 0.35);
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 200;
    backdrop-filter: blur(2px);
}

.auth-card {
    background: #fff;
    border: 1px solid #eaeaea;
    border-radius: 16px;
    padding: 2rem;
    width: 100%;
    max-width: 380px;
    box-shadow: 0 4px 24px rgba(0, 0, 0, 0.08);
    text-align: center;
    animation: fadeIn 0.3s ease;
}

h2 {
    margin-bottom: 1.5rem;
    font-weight: 600;
    color: #111;
}

form {
    display: flex;
    flex-direction: column;
    gap: 1rem;
}

label {
    font-size: 0.9rem;
    font-weight: 500;
    color: #444;
    display: block;
    text-align: left;
    margin-bottom: 0.3rem;
}

input {
    width: 100%;
    padding: 10px 12px;
    border-radius: 8px;
    border: 1px solid #ddd;
    background: #fff;
    transition: border 0.2s, box-shadow 0.2s;
}

input:focus {
    border-color: #bbb;
    box-shadow: 0 0 0 2px rgba(0, 0, 0, 0.03);
    outline: none;
}

.primary-btn {
    background: #111;
    color: #fff;
    padding: 10px;
    border-radius: 8px;
    font-weight: 600;
    border: none;
    cursor: pointer;
    transition: background 0.3s ease;
}

.primary-btn:hover {
    background: #333;
}

.toggle {
    margin-top: 1rem;
    color: #666;
    font-size: 0.9rem;
}

.toggle span {
    color: #111;
    cursor: pointer;
    font-weight: 600;
    margin-left: 4px;
}

@keyframes fadeIn {
    from {
        opacity: 0;
        transform: translateY(-8px);
    }

    to {
        opacity: 1;
        transform: translateY(0);
    }
}
</style>
