<template>
  <div class="max-w-4xl items-center mx-auto flex flex-col gap-3">
    <div class="flex flex-wrap gap-3 justify-center" id="bd">
      <!-- <NotEnroll /> -->
      <CardHome v-for="(enrollment, i) in enrollments" :key="i" :course="enrollment.Course">
        <template #additional>
          <div class="pt-3 flex flex-col gap-1">
            <p class="font-semibold">Progress:</p>
            <v-progress-linear :model-value="getProgress(enrollment)" color="amber" height="25">
              {{ getProgress(enrollment) }}%
            </v-progress-linear>
          </div>
        </template>
        <template #action>
          <div class="py-3 px-9">
            <RouterLink :to="`/enrollments/${enrollment.Course.id}`">
              <CustBtn> Continue This Lesson </CustBtn>
            </RouterLink>
          </div>
        </template>
      </CardHome>
    </div>
    <ListPagination :length="numberOfPages" :page="currentPage" />
  </div>
</template>

<style>
#bd {
  border: 0px solid;
}
</style>

<script>
import client from '../api/config'
import CardHome from '../components/CardHome.vue'
import ListPagination from '../components/ListPagination.vue'
import CustBtn from '../components/CustBtn.vue'
import NotEnroll from '../components/NotEnroll.vue'

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
    NotEnroll
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
      return Math.round((+enrollment.Chapter.chapterNo / enrollment.Course.Chapters.length) * 100)
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
