<template>
  <div>
    <b-overlay :show="show" rounded="sm" @shown="onShown" @hidden="onHidden">
      <section id="main-container" class="main-container p-4">
        <div class="container" :aria-hidden="show ? 'ture' : null">
          <div class="row text-center justify-center">
            <div class="col-lg-8">
              <h3 class="section-sub-title">나와 닮은 배우는? 본인의 사진을 업로드해주세요!</h3>
              <div class="col-lg-8 mx-auto">
                <v-file-input label="2MB 이하로 넣어주세요." v-model="files" prepend-icon="mdi-camera" show-size counter>
                </v-file-input>
              </div>
                <b-button ref="show" :disabled="show" @click="sendImages" elevation="6" text>도플갱어 찾아보기</b-button>
            </div>
          </div>
          <div v-if="doppleganger" class="main-container p-4">
            <div class="row justify-content-center">
              <div class="row text-center">
                <div class="col-lg-12">
                  <h3 class="section-sub-title">{{ username }}님과 {{ doppleganger.celeb}}님은 {{ doppleganger.confidence }} 닮았습니다! <b-icon icon="emoji-wink" font-scale="1.5"></b-icon><b-icon icon="hand-thumbs-up"></b-icon></h3> 
                </div>
              </div>
              <div class="col-lg-3 col-sm-6 mb-5">
                <div class="sc-wrapper">
                  <div class="sc-img-wrapper">
                    <v-img v-if="Selfimgsrc" :src="Selfimgsrc" class="img-fluid" alt="유저이미지"></v-img>
                    <v-img v-else-if="selfurl" :src="selfurl" class="img-fluid" alt="유저이미지"></v-img>
                  </div>
                  <div class="sc-content justify-content-center">
                    <h3>{{ username }}</h3>
                  </div>
                </div>
              </div>
              <div class="col-lg-3 col-sm-6 mb-5">
                <div class="sc-wrapper">
                  <div class="sc-img-wrapper">
                    <img :src="doppleganger.celeb_image" class="img-fluid" alt="배우이미지">
                  </div>
                  <div class="sc-content justify-content-center">
                    <h3>{{ doppleganger.celeb }}</h3>
                  </div>
                </div>
              </div>
            </div>
            <div v-if="doppleganger.celeb_movie_id.length">
              <div class="row text-center">
                <div class="col-lg-12">
                  <h3 class="section-sub-title">{{ doppleganger.celeb}}의 출연작 보러가기</h3>
                </div>
              </div>
            </div>
            <div v-else>
              <div class="row text-center">
                <div class="col-lg-12">
                  <h3 class="section-sub-title">{{ doppleganger.celeb}}의 출연작을 찾을 수 없습니다. <b-icon icon="emoji-frown" font-scale="1.5"></b-icon></h3>
                </div>
              </div>
            </div>
            <div class="row justify-content-center">
              <div  v-for="(poster, idx) in doppleganger.celeb_movie_poster" :key="idx" class="col-lg-3 col-md-4 col-sm-6 mb-5">
                <div class="sc-wrapper">
                  <div class="sc-img-wrapper">
                    <img :src="poster" alt="poster" class="img-fluid" @click="moveToMovieDetail(doppleganger.celeb_movie_id[idx])" >
                  </div>
                  <div class="sc-content justify-content-center">
                    <h3>{{ doppleganger.celeb_movie_title[idx] }}</h3>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </section>
      <template #overlay>
          <div class="text-center">
          <b-icon  icon="stopwatch" font-scale="3" animation="cylon"></b-icon>
          <p id="cancel-label">Please wait...</p>
        </div>
      </template>
    </b-overlay>
  </div>
</template>

<script>
import axios from 'axios'
import { mapState } from 'vuex'
const SERVER_URL = process.env.VUE_APP_SERVER_URL

export default {
  name:'Doppleganger',
  data() {
    return {
      files: null,
      Selfimgsrc: null,
      selfurl : null,
      msg : null,
      doppleganger: null,
      username: null,
      show: false,
    }
  },
  methods: {
    onHidden() {
      // this.$refs.show.focus()
      // this.show=true
      console.log('되고잇나??', this.show)
    },
    onShown() {
      // this.doppleganger
    },
    sendImages() {
      this.show = true
      this.onHidden()
      this.doppleganger = null
      this.msg = null

      let info = new FormData()
      info.append('files', this.files)
      if (this.files===null) {             // 파일을 보내지 않을 경우
        info.append('files', [])
      } else {
        for (let i = 0; i < this.files.length; i++) {   // 파일이 하나 이상인 경우
          info.append('files', this.files[i]);
        }
      }
      const token = localStorage.getItem('JWT') // token을 세션에 저장시켜 사용했기 때문에
      let config = {
        headers: {
          'Content-Type': 'multipart/form-data', // Content-Type을 변경해야 파일이 전송됨
          'Authorization': `Bearer ${token}`
          },
      }
      axios.post(
        `${SERVER_URL}/doppleganger/images/`, 
        info, config)
      .then((res) => {
        console.log(res.data) // 필요한 것 넣어서 쓰면됨
        let img_path = res.data.upload_image
        console.log(`${SERVER_URL}${img_path}`)
        this.Selfimgsrc = `${SERVER_URL}${img_path}`

        this.selfurl = URL.createObjectURL(this.files)
        console.log(this.selfurl)
        this.$store.dispatch('saveSelfurl', this.selfurl) //프로필에서 쓸 이미지 vuex로 보내기        
        this.FindDopplegangerInfo()
      })
      .catch((err) => {
        console.log('이미지가 없음')
        console.log(err)
      })
    },

    setHeader: function() {
      const token = localStorage.getItem('JWT')
      const header = {
        Authorization : `Bearer ${token}`
      }
      return header
    },

    FindDopplegangerInfo: function () {
      axios({
        mehtod: 'get',
        url:`${SERVER_URL}/doppleganger/`,
        headers: this.setHeader()
      })
      .then(res => {
        console.log(res.data)
        if (res.data.noImg) {
          this.msg = '아직 사진을 올리지 않았어요! 사진을 올려주세요.'
        } else if (res.data.celeb_movie_id.length === 0) {
          this.doppleganger = res.data
          this.username = res.data.upload_username
          this.msg = '이 배우는 찍은 필모그래피가 없어요!'
          this.show = false
          this.onShown()
        }        
        else {
          this.doppleganger = res.data
          this.username = res.data.upload_username
          // this.$store.dispatch('saveDoppleganger', this.doppleganger)
          this.msg = null
          this.show = false
          this.onShown()
        }
      })
      .catch(err => {
        console.log(err)
        console.log('닮은 배우가 없음.')
      })
    },
    moveToMovieDetail: function (movie_id) {
     this.$router.push({name: 'MovieDetail', params: {'movie_num':movie_id}})
   },
  },
  created () {
    // this.getDoppleganger()
  },
  computed: {
      ...mapState(['UserId'])
    }      
}
</script>

<style scoped>
.h3.section-sub-title {
  font-family: 'Montserrat', sans-serif;
}
.card-list {
  margin-top: 3rem;
  margin-left: 0rem;
  margin-right: 0rem;
  justify-content: center;
  width: 100%;
}
.dopple {
  justify-content: center;
}
</style>