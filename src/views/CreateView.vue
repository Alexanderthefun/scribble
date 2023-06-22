<script setup>
import { Timestamp, addDoc } from 'firebase/firestore'
import Button from 'primevue/button'
import Chips from 'primevue/chips'
import InputText from 'primevue/inputtext'
import TextArea from 'primevue/textarea'
import { ref } from 'vue'
import { postsCollection } from '../firebase'
import { useRouter } from 'vue-router'

const form = ref({
  title: '',
  subtitle: '',
  content: '',
  imageUrl: 'https://picsum.photos/600/400', // TODO
  tags: [],
  publishDate: null,
  readTime: 0,
  authorId: 'some_author_id', // TODO
  authorAvatar: 'https://robohash.org/undequamanimi.png?size=50x50&set=set1', // TODO
  authorName: 'Luke Skywalker' // TODO
})

const router = useRouter()

const onSubmit = async () => {
  form.value.publishDate = Timestamp.now()

  const totalWords = form.value.content.split(' ').length
  const readTime = Math.floor(totalWords / 250)
  form.value.readTime = readTime

  const response = await addDoc(postsCollection, form.value)

  router.push(`/post/${response.id}`)
}
</script>

<template>
  <div>
    <form @submit.prevent="onSubmit">
      <div class="flex flex-column mt-6">
        <InputText v-model="form.title" placeholder="Title" class="mb-2" />
        <InputText v-model="form.subtitle" placeholder="Subtitle" class="mb-2" />
        <Chips v-model="form.tags" placeholder="Tags" class="mb-2" />
        <TextArea v-model="form.content" auto-resize rows="30" class="mb-2" />
      </div>

      <Button type="submit">Publish</Button>
    </form>
  </div>
</template>