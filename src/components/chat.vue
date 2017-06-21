<template>
<div id="chat">
<div v-for="msg in messages">{{msg.message}}</div>
        <input type="text" v-model.trim="messageInput" @keyup.enter="send(messageInput)">
    </div>
</template>
<script>
  import * as Firebase from 'firebase'
  import Geohash from 'latlon-geohash'
  export default {
    name: 'chat',
    methods: {
      init () {
        if (navigator.geolocation) {
          navigator.geolocation.getCurrentPosition((position) => {
            var geohash = Geohash.encode(position.coords.latitude, position.coords.longitude, this.precision)

            this.room = this.db.database().ref().child('rooms/' + geohash)
            console.log(this.room)
            this.messageListener()
          }, (err) => {
            console.log(err)
          })
        } else {
          console.error('Cannot access geolocation')
        }
      },
      send (messageInput) {
      // A data entry.
        let data = {
          message: messageInput
        }

    // Get a key for a new message.
        let key = this.room.push().key
        this.room.child('messages/' + key).set(data)
  
      // clean the message
        this.messageInput = ''
      },
      messageListener () {
        this.room.child('messages').on('child_added', (snapshot) => {
        // push the snapshot value into a data attribute
          this.messages.push(snapshot.val())
        })
      }
    },
    data () {
      return {
        room: null,
        precision: 6,
        db: null,
        messageInput: '', // this is for v-model
        messages: []
      }
    },
    mounted () {
      this.db = Firebase.initializeApp({
        apiKey: 'AIzaSyAXsSMWUMwokHLXaokDejgfwNkGkxTyhV8',
        authDomain: 'chatroom-7f389.firebaseapp.com',
        databaseURL: 'https://chatroom-7f389.firebaseio.com',
        projectId: 'chatroom-7f389',
        storageBucket: 'chatroom-7f389.appspot.com',
        messagingSenderId: '7478357589'
      })
      console.log(this.db)
      this.init()
    }
  }
</script>
<style lang="css" scoped>
</style>
