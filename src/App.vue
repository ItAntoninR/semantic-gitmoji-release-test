<script setup lang="ts">
import { ref } from 'vue'
import * as jose from 'jose'

const token = ref<string | null>(null)
const kid = ref<string | null>(null)

const loadPrivateKeyAndGenerateKid = async () => {
  try {
    const privateKeyPem = await (await fetch('/keys/jws_private.pem')).text()

    if (!privateKeyPem) {
      throw new Error('Clé privée non trouvée')
    }

    const privateKey = await jose.importPKCS8(privateKeyPem, 'RS256')

    kid.value = privateKey.kid || 'default-kid'

    return { privateKey, kid: kid.value }
  } catch (error) {
    console.error('Erreur lors du chargement de la clé privée:', error)
    throw error
  }
}

const signDataAndSend = async () => {
  try {
    const data = {
      name: 'BOUSNINA Achraf',
      email: 'bsachref@gmail.com',
    }

    const { privateKey, kid } = await loadPrivateKeyAndGenerateKid()

    token.value = await new jose.SignJWT(data)
      .setProtectedHeader({
        alg: 'RS256',
        kid: kid as string,
      })
      .setIssuedAt()
      .setExpirationTime('1h')
      .sign(privateKey)

    console.log('JWT signé:', token.value)
  } catch (error) {
    console.error('Erreur lors de la signature:', error)
  }
}
</script>

<template>
  <div>
    <button @click="signDataAndSend">Signer et envoyer</button>
    <div v-if="token">
      <h3>JWT :</h3>
      <pre>{{ token }}</pre>
    </div>
    <div v-if="kid">
      <h3>KID :</h3>
      <pre>{{ kid }}</pre>
    </div>
  </div>
</template>

<style scoped>
button {
  padding: 10px 20px;
  margin-bottom: 20px;
  font-size: 16px;
}
pre {
  background: #f4f4f4;
  padding: 10px;
  border: 1px solid #ddd;
}
</style>
