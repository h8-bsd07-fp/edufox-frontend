<template>
  <div class="max-w-4xl items-center mx-auto flex flex-col gap-3 p-8">
    <Unauthorized v-if="!isLoggedIn" />
    <div v-else class="flex flex-wrap gap-3 justify-center">
      <div v-if="enrollments.length === 0" class="py-24">
        <div class="max-w-sm">
          <img
            class="object-cover"
            src="https://img.icons8.com/external-becris-lineal-becris/512/acacac/external-book-literary-genres-becris-lineal-becris.png"
            alt="external-book-literary-genres-becris-lineal-becris"
          />
        </div>
        <p class="text-[#7b7b7b] text-center text-lg">
          <RouterLink to="/courses" class="underline text-orange-400">Click here</RouterLink> to add
          courses to your study list
        </p>
      </div>
      <CardHome v-else v-for="(enrollment, i) in enrollments" :key="i" :course="enrollment.Course">
        <template #additional>
          <div class="pt-3 flex flex-col gap-1">
            <p class="font-semibold">Progress:</p>
            <v-progress-linear
              :model-value="isFinished(enrollment) ? 100 : getProgress(enrollment)"
              color="amber"
              height="25"
            >
              {{ `${finishedChapters(enrollment)} of ${totalChapters(enrollment)} Chapter(s)` }}
            </v-progress-linear>
          </div>
        </template>
        <template #action>
          <div class="py-3">
            <RouterLink :to="`/enrollments/${enrollment.Course.id}/${enrollment.curChapterId}`">
              <CustBtn>
                {{
                  isFinished(enrollment)
                    ? 'Review'
                    : isOnProgress(enrollment)
                      ? 'Continue This Lesson'
                      : 'Start Learning'
                }}
              </CustBtn>
            </RouterLink>
          </div>
        </template>
      </CardHome>
    </div>
    <ListPagination :length="numberOfPages" :page="currentPage" />
  </div>
</template>

<script>
import { RouterLink } from 'vue-router'
import { mapState } from 'pinia'
import useUserStore from '../stores/user'
import client from '../api/config'
import CardHome from '../components/CardHome.vue'
import ListPagination from '../components/ListPagination.vue'
import CustBtn from '../components/CustBtn.vue'
import Unauthorized from '../components/Unauthorized.vue'

export default {
  name: 'EnrollmentsView',
  data() {
    return {
      enrollments: [],
      numberOfPages: 0,
      currentPage: this.query.page || 1
    }
  },
  props: ['query'],
  components: {
    CardHome,
    ListPagination,
    CustBtn,
    Unauthorized,
    RouterLink
  },
  computed: {
    ...mapState(useUserStore, ['isLoggedIn'])
  },
  methods: {
    getEnrollments() {
      const query = new URLSearchParams(this.query).toString()
      client
        .get(`/enrollments${query && `?${query}`}`, {
          headers: {
            access_token: localStorage.getItem('access_token')
          }
        })
        .then(({ data }) => {
          this.enrollments = data.data
          this.numberOfPages = data.totalPage
          this.currentPage = +data.currentPage
        })
        .catch((err) => console.log(JSON.stringify(err, null, 4)))
    },
    getProgress(enrollment) {
      return Math.round(
        ((+enrollment.Chapter.chapterNo - 1) / this.totalChapters(enrollment)) * 100
      )
    },
    isOnProgress(enrollment) {
      return enrollment.status === 'on progress'
    },
    isFinished(enrollment) {
      return enrollment.status === 'finished'
    },
    totalChapters(enrollment) {
      return enrollment.Course.Chapters.length
    },
    finishedChapters(enrollment) {
      return this.isFinished(enrollment)
        ? this.totalChapters(enrollment)
        : +enrollment.Chapter.chapterNo - 1
    }
  },
  watch: {
    query() {
      this.getEnrollments()
    }
  },

  created() {
    this.getEnrollments()
  }
}
</script>
