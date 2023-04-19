<script setup lang="ts">
import { ref } from 'vue';
import InputText from 'primevue/inputtext';
import {
  Configuration,
  OpenAIApi,
  ChatCompletionRequestMessageRoleEnum,
} from 'openai'
import type { ChatCompletionResponseMessage } from 'openai'

const configuration = new Configuration({
    apiKey: import.meta.env.VITE_OPENAI_API_KEY,
  })
const openai = new OpenAIApi(configuration)

const prompt = ref('');
const messages = ref<ChatCompletionResponseMessage[]>([]);
// const messages = ref<ChatCompletionResponseMessage[]>([{ role: ChatCompletionRequestMessageRoleEnum.System, content: '語尾にぴょんを付けて'}]);
/** 
 * [
 *   { role: 'user', content: 'こんにちは' },
 *   { role: 'assistant', content: 'こんにちは！お元気ですか？' },
 *   { role: 'user', content: '元気です"' },
 *   { role: 'assistant', content: 'それは良かったですね。何かお話しましょうか？' },
 * ]
 */

const enter = async () => {
  await createChatCompletion(prompt.value)
}

const createChatCompletion = async (content: string) => {
  messages.value.push({
    role: ChatCompletionRequestMessageRoleEnum.User,
    content,
  })

  const response = await openai.createChatCompletion({
    model: 'gpt-3.5-turbo',
    messages: messages.value,
  })

  messages.value.push({
    role: ChatCompletionRequestMessageRoleEnum.Assistant,
    content: response.data.choices[0].message ? response.data.choices[0].message.content : ''
  })
}

</script>

<template>
  <main>
    <InputText @keyup.enter="enter" class="w-full" id="prompt" v-model="prompt" />
    <div
      :style="{
        textAlign:
          message.role === ChatCompletionRequestMessageRoleEnum.User
            ? 'right'
            : 'left',
        display: message.role === ChatCompletionRequestMessageRoleEnum.System ? 'none' : 'block',
      }"
      v-for="message in messages"
    >
      {{ message }}
    </div>
  </main>
</template>

<style scoped>

</style>
