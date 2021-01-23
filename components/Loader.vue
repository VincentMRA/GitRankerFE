<template>
    <v-row justify="center" align="center" >
        <div class="loader"></div>
    </v-row>    
</template>
<style>
.loader {
  border: 16px solid #808080; /* Light grey */
  border-top: 16px solid hsl(189, 70%, 53%); /* Blue */
  border-radius: 50%;
  width: 120px;
  height: 120px;
  animation: spin 2s linear infinite;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}
</style>