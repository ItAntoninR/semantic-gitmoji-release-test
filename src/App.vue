<script setup lang="ts">
import { ref } from 'vue'
import * as jose from 'jose'

const token = ref<string | null>(null)
const kid = ref<string | null>(null)

// Fonction pour charger la clé privée et générer le kid
const loadPrivateKeyAndGenerateKid = async () => {
  try {
    // Charger la clé privée depuis le dossier public
    const privateKeyPem = await (await fetch('/keys/jws_private.pem')).text()

    if (!privateKeyPem) {
      throw new Error('Clé privée non trouvée')
    }

    // Importer la clé privée au format PKCS8
    const privateKey = await jose.importPKCS8(privateKeyPem, 'RS256')

    // Extraire le `kid` directement de la clé
    kid.value = privateKey.kid || 'default-kid-test-release' // Si le `kid` n'est pas défini, on utilise un kid par défaut.

    return { privateKey, kid: kid.value }
  } catch (error) {
    console.error('Erreur lors du chargement de la clé privée:', error)
    throw error
  }
}

// Fonction pour signer les données
const signDataAndSend = async () => {
  try {
    const data = {
      name: 'BOUSNINA Achraf',
      email: 'bsachref@gmail.com',
    }

    // Charger la clé privée et le kid
    const { privateKey, kid } = await loadPrivateKeyAndGenerateKid()

    // Signer le JWT
    token.value = await new jose.SignJWT(data)
      .setProtectedHeader({
        alg: 'RS256',
        kid: kid as string,
      })
      .setIssuedAt()
      .setExpirationTime('1h')
      .sign(privateKey)

    console.log('JWT signé:', token.value)
    alert(`JWT signé:\n${token.value}`)
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
