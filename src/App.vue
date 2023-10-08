<template>
  <div class="flex flex-col h-screen bg-zinc-100 dark:bg-zinc-900">
    <div class="flex items-center justify-center h-16 px-4 bg-white dark:bg-zinc-800 shadow-md">
      <h1 class="text-xl font-bold text-zinc-800 dark:text-zinc-200">Плейлист</h1>
    </div>
    <div class="flex flex-col flex-1 p-8">
      <div class="flex items-center justify-center mb-12">
        <img
          class="w-96 max-h-96 rounded-lg shadow-lg"
          :src="uri + currentTrack.coverStorage"
          alt="Обложка трека"
        />
      </div>
      <div class="flex flex-col items-center mb-20">
        <h2 class="text-2xl font-semibold text-zinc-800 dark:text-zinc-200">
          {{ currentTrack.title }}
        </h2>
        <p class="text-sm text-zinc-600 dark:text-zinc-400">{{ currentTrack.artist }}</p>
      </div>
      <div class="flex w-full items-center justify-evenly sm:justify-center sm:space-x-10">
        <div class="relative">
          <IconButton @click="toggleVolume" class="block">
            <Icon size="32">
              <template v-if="isMuted">
                <VolumeMute />
              </template>
              <template v-else-if="volume > 75">
                <VolumeHighOutline />
              </template>
              <template v-else-if="volume > 25 && volume <= 75">
                <VolumeMediumOutline />
              </template>
              <template v-else-if="volume <= 25">
                <VolumeLowOutline />
              </template>
            </Icon>
          </IconButton>
          <!-- Недослайдер -->
          <div v-if="showVolume && !isMuted" class="absolute -top-14 -right-6">
            <input
              type="range"
              min="1"
              max="100"
              v-model="volume"
              @input="changeVolume"
              class="bg-black rounded-md appearance-none cursor-pointer dark:bg-white -rotate-90 w-20"
            />
          </div>
        </div>
        <IconButton @click="prevTrack">
          <Icon size="32">
            <PlaySkipBack />
          </Icon>
        </IconButton>
        <IconButton @click="togglePlay">
          <Icon size="64">
            <template v-if="isPlaying">
              <PauseCircle />
            </template>
            <template v-else>
              <PlayCircle />
            </template>
          </Icon>
        </IconButton>
        <IconButton @click="nextTrack">
          <Icon size="32">
            <PlaySkipForward />
          </Icon>
        </IconButton>
        <IconButton @click="toggleMute">
          <Icon size="32">
            <VolumeOff
              class="rounded-full"
              :class="
                isMuted ? 'text-red-600 transition-colors duration-150 hover:text-red-400' : ''
              "
            />
          </Icon>
        </IconButton>
      </div>
    </div>
    <audio
      ref="audio"
      :src="uri + currentTrack.musicStorage"
      type="audio/mp3"
      @ended="nextTrack"
    ></audio>
    <PageLoader v-if="isFetching" />
  </div>
</template>

<script>
import { computed, onMounted, ref } from 'vue'
import {
  PauseCircle,
  PlayCircle,
  PlaySkipBack,
  PlaySkipForward,
  VolumeHighOutline,
  VolumeLowOutline,
  VolumeMediumOutline,
  VolumeMute,
  VolumeOff
} from '@vicons/ionicons5'

import { Icon } from '@vicons/utils'
import IconButton from '@/components/IconButton.vue'
import axios from 'axios'
import PageLoader from './components/PageLoader.vue'

export default {
  components: {
    IconButton,
    PauseCircle,
    PlayCircle,
    PlaySkipForward,
    PlaySkipBack,
    VolumeHighOutline,
    VolumeMediumOutline,
    VolumeLowOutline,
    Icon,
    PageLoader,
    VolumeMute,
    VolumeOff
  },
  setup() {
    const uri = 'https://interview.vobrabotke.ru'
    const token = '886c2a7449f56a783e709de10a91fcc6d7bbe2865120050ad4a3b57b2732c7d9'
    const playlist = ref(null)
    const error = ref(null)
    const isFetching = ref(false)

    const audio = ref(null)
    const isPlaying = ref(false)
    const isMuted = ref(false)
    const showVolume = ref(false)
    const volume = ref(100)
    const fadeDuration = 3000
    const fadeInterval = 250
    const trackFadedIn = ref(false)
    const beforeMute = ref(0)
    const tmpPlaying = ref(null)

    const currentTrackIndex = ref(0)
    const currentTrack = computed(() => {
      if (playlist.value && playlist.value.data) {
        return {
          title: playlist.value.data.music[currentTrackIndex.value].title,
          artist: playlist.value.data.music[currentTrackIndex.value].artist,
          musicStorage: playlist.value.data.music[currentTrackIndex.value].musicStorage,
          coverStorage: playlist.value.data.music[currentTrackIndex.value].coverStorage
        }
      } else {
        return {
          title: '',
          artist: '',
          musicStorage: '',
          coverStorage: ''
        }
      }
    })

    const togglePlay = () => {
      if (isPlaying.value) {
        pauseTrack()
      } else {
        playTrack()
      }
    }

    const playTrack = () => {
      isPlaying.value = true
      audio.value.play().catch((err) => {
        console.error(err)
        error.value = err.message
      })
      fadeIn()

      const duration = audio.value.duration
      const delay = (duration - (fadeDuration/1000) )
      setTimeout(fadeOut, delay)
      console.log(`track duration: ${duration} <=> delay: ${delay}`)
    }

    const pauseTrack = () => {
      isPlaying.value = false
      audio.value.pause()
    }

    const nextTrack = () => {
      if (playlist.value && playlist.value.data) {
        trackFadedIn.value = false
        tmpPlaying.value = isPlaying.value
        currentTrackIndex.value = (currentTrackIndex.value + 1) % playlist.value.data.music.length
        audio.value.load()
        pauseTrack()
        if (tmpPlaying.value) {
          setTimeout(() => {
            playTrack()
          })
        }
      }
    }

    const prevTrack = () => {
      if (playlist.value && playlist.value.data) {
        trackFadedIn.value = false
        tmpPlaying.value = isPlaying.value
        currentTrackIndex.value =
          (currentTrackIndex.value - 1 + playlist.value.data.music.length) %
          playlist.value.data.music.length
        audio.value.load()
        pauseTrack()
        if (tmpPlaying.value) {
          setTimeout(() => {
            playTrack()
          })
        }
      }
    }

    const toggleVolume = () => {
      showVolume.value = !showVolume.value
    }

    const changeVolume = () => {
      audio.value.volume = volume.value / 100
    }

    const toggleMute = () => {
      isMuted.value = !isMuted.value
      if (isMuted.value) {
        audio.value.muted = true
        beforeMute.value = volume.value
        volume.value = 0
      } else {
        audio.value.muted = false
        volume.value = beforeMute.value
      }
    }

    const fadeIn = () => {
      if (!trackFadedIn.value) {
        // если трек не запущен
        const targetVolume = volume.value
        volume.value = 0
        const numberOfFades = fadeDuration / fadeInterval
        const deltaVolume = Math.round(100 / numberOfFades)
        const fade = () => {
          if (volume.value < 100) {
            if (volume.value + deltaVolume > targetVolume) {
              volume.value = targetVolume
              return
            }
            volume.value += deltaVolume
            audio.value.volume = volume.value / 100
            setTimeout(fade, 300)
            return
          } else {
            volume.value = audio.value.volume
            return
          }
        }
        fade()
        trackFadedIn.value = true
      } else {
        return
      }
    }

    const fadeOut = () => {
      if (trackFadedIn.value) {// если трек запущен
        const targetVolume = 0
        const numberOfFades = fadeDuration / fadeInterval
        const deltaVolume = Math.round(100 / numberOfFades)
        const fade = () => {
          if (volume.value > targetVolume) {
            if (volume.value - deltaVolume < targetVolume) {
              volume.value = targetVolume
              return
            }
            volume.value -= deltaVolume
            audio.value.volume = volume.value / 100
            setTimeout(fade, 300)
          } else {
            volume.value = audio.value.volume
            return
          }
        }
        fade()
      } else {
        return
      }
    }

    onMounted(() => {
      audio.value.volume = volume.value / 100
      isFetching.value = true
      axios
        .get('https://interview.vobrabotke.ru/player/api/playlist/0', {
          headers: {
            Authorization: `Bearer ${token}`
          }
        })
        .then((response) => {
          playlist.value = response.data
          isFetching.value = false
        })
        .catch((err) => {
          error.value = err.message
          isFetching.value = false
        })
    })

    return {
      playlist,
      error,
      isFetching,
      audio,
      isPlaying,
      isMuted,
      showVolume,
      volume,
      fadeInterval,
      fadeDuration,
      trackFadedIn,
      currentTrackIndex,
      currentTrack,
      togglePlay,
      playTrack,
      pauseTrack,
      nextTrack,
      prevTrack,
      toggleVolume,
      changeVolume,
      toggleMute,
      uri,
      fadeIn,
      fadeOut
    }
  }
}
</script>

<style scoped></style>
