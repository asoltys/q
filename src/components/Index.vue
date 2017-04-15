<template>
  <q-layout>
    <div slot="header" class="toolbar">
      <q-toolbar-title :padding="0">
        Quasar Framework v{{$q.version}}
      </q-toolbar-title>
      <button @click="clear">Clear</button>
    </div>

    <!--
      Replace following "div" with
      "<router-view class="layout-view">" component
      if using subRoutes
    -->
    <div class="layout-view" @click='vibrate'>
      <div class="logo-container non-selectable no-pointer-events">
        <div class="logo" :style="position">
          <img src="~assets/quasar-logo.png">
          <p class="caption text-center">
            <span class="desktop-only">Move your mouse.</span>
            <span class="touch-only">Touch screen and move.</span>
            <p>{{message}}</p>
            <ul v-for='book in books'>
              <li>{{ book.title }}</li>
            </ul>
          </p>
        </div>
      </div>
    </div>
  </q-layout>
</template>

<script>
  import * as ds from 'deepstream.io-client-js'

  var moveForce = 30
  var rotateForce = 40

  import { Utils } from 'quasar'
  export default {
    data () {
      return {
        moveX: 0,
        moveY: 0,
        rotateY: 0,
        rotateX: 0,
        message: '',
        books: [],
        book: {
          title: '',
          year: '',
          author: '',
          read: false
        },
        ds: ds('192.168.10.251:6020')
      }
    },
    created () {
      this.ds.login({}, () => {
        console.log('logged in')
      })
      this.ds.record.getList('books').subscribe((books) => {
        this.books = books.map((record) => {
          this.ds.record.getRecord(record)
        })
        console.log(this.books)
      })
    },
    computed: {
      position () {
        let transform = `rotateX(${this.rotateX}deg) rotateY(${this.rotateY}deg)`
        return {
          top: this.moveY + 'px',
          left: this.moveX + 'px',
          '-webkit-transform': transform,
          '-ms-transform': transform,
          transform
        }
      }
    },
    methods: {
      clear () {
        this.ds.record.getList('books').delete()
      },
      move (event) {
        const {width, height} = Utils.dom.viewport()
        const {top, left} = Utils.event.position(event)
        const halfH = height / 2
        const halfW = width / 2

        this.moveX = (left - halfW) / halfW * -moveForce
        this.moveY = (top - halfH) / halfH * -moveForce
        this.rotateY = (left / width * rotateForce * 2) - rotateForce
        this.rotateX = -((top / height * rotateForce * 2) - rotateForce)
      },
      vibrate () {
        navigator.vibrate(5000)
        this.message = 'oh yeah ' + (new Date()).toString()
        const recordName = this.book.id || 'book/' + this.ds.getUid()

        this.ds.record.has(recordName, (err, has) => {
          if (err) {
            console.log(err)
            return
          }
          else {
            console.log('here')

            this.book = {
              title: 'Lord of the Rings',
              year: '',
              author: '',
              read: false
            }

            const bookRecord = this.ds.record.getRecord(recordName)
            bookRecord.set(this.book)
            console.log(bookRecord)

            this.ds.record.getList('books').addEntry(recordName)
          }
        })
      }
    },
    mounted () {
      this.$nextTick(() => {
        document.addEventListener('mousemove', this.move)
        document.addEventListener('touchmove', this.move)
      })
    },
    beforeDestroy () {
      document.removeEventListener('mousemove', this.move)
      document.removeEventListener('touchmove', this.move)
    }
  }
</script>

<style lang="stylus">
.logo-container
  width 192px
  height 268px
  perspective 800px
  position absolute
  top 50%
  left 50%
  transform translateX(-50%) translateY(-50%)
.logo
  position absolute
  transform-style preserve-3d
</style>
