<script setup>
import { Timestamp, addDoc } from 'firebase/firestore'
import Button from 'primevue/button'
import Chips from 'primevue/chips'
import InputText from 'primevue/inputtext'
import TextArea from 'primevue/textarea'
import { ref } from 'vue'
import { postsCollection, storage } from '../firebase'
import { useRouter } from 'vue-router'
import { user, username } from '../stores/authStore'
import FileUpload from 'primevue/fileupload'
import { ref as getStorageRef, uploadBytes, getDownloadURL } from 'firebase/storage'

const form = ref({
  title: '',
  subtitle: '',
  content: '',
  imageUrl: null, // TODO
  tags: [],
  publishDate: null,
  readTime: 0,
  authorId: user.value.uid,
  authorAvatar: user.value.photoURL,
  authorName: username.value
})

const router = useRouter()

const onSubmit = async () => {
  if (image.value) {
    const originalFilename = image.value.name
    const fileExtension = originalFilename.split('.').pop()
    const newFileName = `${Date.now()}.${fileExtension}`

    // This is a reference to where in Firebase this particular file will be stored
    const imageRef = getStorageRef(storage, newFileName)
    await uploadBytes(imageRef, image.value)
    const newImageUrl = await getDownloadURL(imageRef)

    form.value.imageUrl = newImageUrl
  }

  form.value.publishDate = Timestamp.now()

  const totalWords = form.value.content.split(' ').length
  const readTime = Math.floor(totalWords / 250)
  form.value.readTime = readTime

  const response = await addDoc(postsCollection, form.value).then((response) => {
    router.push(`/post/${response.id}`)
  })
}

const image = ref(null)

const onFileSelect = (event) => {
  image.value = event.files[0]
}
</script>

<template>
  <div>
    <form @submit.prevent="onSubmit">
      <div class="flex flex-column mt-6">
        <InputText v-model="form.title" placeholder="Title" class="mb-2" />
        <InputText v-model="form.subtitle" placeholder="Subtitle" class="mb-2" />
        <Chips v-model="form.tags" placeholder="Tags" class="mb-2" />
        <FileUpload
          custom-upload
          accept="image/*"
          :max-file-size="1_000_000"
          choose-label="Choose Image"
          mode="basic"
          class="mb-2"
          @select="onFileSelect"
        />
        <TextArea v-model="form.content" auto-resize rows="30" class="mb-2" />
      </div>

      <Button type="submit">Publish</Button>
    </form>
  </div>
</template>
