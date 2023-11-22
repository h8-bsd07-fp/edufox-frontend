<template>
  <div class="max-w-4xl items-center mx-auto flex flex-col gap-3">
    <div class="flex flex-wrap gap-3 justify-center">
      <CardHome v-for="(enrollment, i) in enrollments" :key="i" :course="enrollment.Course">
        <template #additional>
          <div class="pt-3 flex flex-col gap-1">
            <p class="font-semibold">Progress:</p>
            <v-progress-linear
              :model-value="isFinished(enrollment) ? 100 : getProgress(enrollment)"
              color="amber"
              height="25"
            >
              {{ isFinished(enrollment) ? 100 : getProgress(enrollment) }}%
            </v-progress-linear>
          </div>
        </template>
        <template #action>
          <div class="py-3 px-6">
            <RouterLink :to="`/enrollments/${enrollment.Course.id}/${enrollment.curChapterId}`">
              <CustBtn>
                {{
                  enrollment.status === 'created'
                    ? 'Start Learning'
                    : isFinished(enrollment)
                      ? 'Review'
                      : 'Continue This Lesson'
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
import client from '../api/config'
import CardHome from '../components/CardHome.vue'
import ListPagination from '../components/ListPagination.vue'
import CustBtn from '../components/CustBtn.vue'

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
    CustBtn
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

          // console.log(data)
          // console.log(this.numberOfPages)
        })
        .catch((err) => console.log(JSON.stringify(err, null, 4)))
    },
    getProgress(enrollment) {
      return Math.round(
        ((+enrollment.Chapter.chapterNo - 1) / enrollment.Course.Chapters.length) * 100
      )
    },
    isFinished(enrollment) {
      return enrollment.status === 'finished'
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
