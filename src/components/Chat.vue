<template>
	<div class="q-pa-md row justify-center">
		<div class="message-content">
			<q-chat-message v-for="(item, index) in messages" :key="index" :name="item.name" :text="[item.content]"
				:sent="item.name === 'ME'">

				<template v-slot:avatar>
					<q-avatar v-if="item.name === 'ME'" color="orange">Me</q-avatar>
					<q-avatar v-else>
						<img src="../assets/openai.jpeg" alt="GPT" />
					</q-avatar>
				</template>
			</q-chat-message>

			<q-chat-message v-show="sending" name="GPT" avatar="../assets/openai.jpeg">
				<q-spinner-dots size="1.5em" />
			</q-chat-message>
		</div>
	</div>

	<q-input class="input-text" ref="inputRef" v-model="text" placeholder="say something..." filled autogrow
		@keypress.enter.prevent="sendMessage">
		<template v-slot:append>
			<q-btn flat round color="primary" icon="send" :disabled="!text.trim()" @click.prevent="sendMessage" />
		</template>
	</q-input>
</template>

<script setup>
	import {
		ref
	} from 'vue'

	import {
		Configuration,
		OpenAIApi
	} from "openai";
	import {
		useQuasar
	} from 'quasar'

	const $q = useQuasar();

	const key = 'c2steG9BREZQdU1GMTFYdWpQNk9BZEdUM0JsYmtGSlZBaGh2NUVBN2FjREdmcEJ3Y3JU';

	const configuration = new Configuration({
		apiKey: window.atob(key)
	});
	const openai = new OpenAIApi(configuration);

	const text = ref('')
	const messages = ref([]);
	const sending = ref(false);
	let lastText = '';

	// 发送请求
	const sendMessage = async () => {
		if (sending.value) {
			$q.notify({
				type: 'warning',
				message: 'Please wait a moment'
			});
			return;
		}
		sending.value = true;
		messages.value.push({
			name: 'ME',
			content: text.value
		});
		lastText = text.value;
		text.value = '';

		setTimeout(() => {
			window.scrollTo(0, document.documentElement.scrollHeight);
		}, 100)

		const completion = await openai.createChatCompletion({
			model: "gpt-3.5-turbo",
			messages: [{
				role: "user",
				content: lastText
			}]
		}).catch(err => {
			$q.notify({
				type: 'warning',
				message: 'Please wait a moment'
			});
			messages.value.shift();
		});

		sending.value = false;

		if (!completion || !completion.data) {
			return;
		}

		messages.value.push({
			name: 'GPT',
			content: completion.data.choices[0].message.content
		});

		setTimeout(() => {
			window.scrollTo(0, document.documentElement.scrollHeight);
		}, 100)

	}
</script>


<style type="text/css" lang="scss">
	.input-text {
		width: 100%;
		position: fixed;
		left: 0;
		bottom: 0;
		background-color: white;
	}

	.q-message-text-content {
		min-width: 5em;
		line-height: 1.5em;
	}

	.q-message-text:last-child {
		min-height: 0;
	}

	.q-scroll {
		width: 100% !important;
		max-height: 100%;
	}

	.q-message-received {
		.q-avatar {
			margin-right: 5px;
		}
	}

	.q-message-sent {
		.q-avatar {
			margin-left: 5px;
		}
	}

	.message-content {
		width: 100%;
		padding-bottom: 60px;
	}
</style>