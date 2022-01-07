<template>
	<div id="register" class="window" role="tabpanel" aria-label="Register">
		<form class="container" method="post" action="" @submit="onSubmit">
			<img
				src="img/logo-vertical-transparent-bg.svg"
				class="logo"
				alt="The Lounge"
				width="256"
				height="170"
			/>
			<img
				src="img/logo-vertical-transparent-bg-inverted.svg"
				class="logo-inverted"
				alt="The Lounge"
				width="256"
				height="170"
			/>

			<label for="register-username">Username</label>
			<input
				id="register-username"
				ref="username"
				class="input"
				type="text"
				name="username"
				autocapitalize="none"
				autocorrect="off"
				autocomplete="username"
				:value="getStoredUser()"
				required
				autofocus
			/>

			<div class="password-container">
				<label for="register-password">Password</label>
				<RevealPassword v-slot:default="slotProps">
					<input
						id="register-password"
						ref="password"
						:type="slotProps.isVisible ? 'text' : 'password'"
						name="password"
						class="input"
						autocapitalize="none"
						autocorrect="off"
						autocomplete="current-password"
						required
					/>
				</RevealPassword>
			</div>

			<div v-if="errorShown" class="error">Username taken.</div>

			<button :disabled="inFlight" type="submit" class="btn">Register</button>
		</form>
	</div>
</template>

<script>
import storage from "../../js/localStorage";
import socket from "../../js/socket";
import RevealPassword from "../RevealPassword.vue";

export default {
	name: "Register",
	components: {
		RevealPassword,
	},
	data() {
		return {
			inFlight: false,
			errorShown: false,
		};
	},
	mounted() {
		socket.on("auth:failed", this.onRegisterFailed);
		socket.on("auth:registered", this.onRegisterSucceed);
	},
	beforeDestroy() {
		socket.off("auth:failed", this.onRegisterFailed);
		socket.off("auth:registered", this.onRegisterSucceed);
	},
	methods: {
		onRegisterFailed() {
			this.inFlight = false;
			this.errorShown = true;
		},
		onRegisterSucceed() {
			this.inFlight = true;
			this.errorShown = false;

			const values = {
				user: this.$refs.username.value,
				password: this.$refs.password.value,
			};
			storage.set("user", values.user);
			socket.emit("auth:perform", values);
		},
		onSubmit(event) {
			event.preventDefault();

			this.inFlight = true;
			this.errorShown = false;

			const values = {
				user: this.$refs.username.value,
				password: this.$refs.password.value,
			};

			storage.set("user", values.user);

			socket.emit("auth:register", values);
		},
		getStoredUser() {
			return storage.get("user");
		},
	},
};
</script>
