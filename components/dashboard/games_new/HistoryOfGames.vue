<template>
  <div>
    <SectionHeader title="تاریخچه بازی ها" icon="mdi-history"/>
    <v-simple-table v-if="data && data.length > 0">
      <template v-slot:default>
        <thead>
        <tr>
          <th class="text-right">
            نام تیم
          </th>
          <th class="text-right">
            تورنومنت
          </th>
          <!--              <th class="text-right">-->
          <!--                اطلاعات-->
          <!--              </th>-->
          <th class="text-center">
            لاگ بازی
          </th>
          <th class="text-center">
            وضعیت
          </th>
          <th class="text-center">
            برنده
          </th>
        </tr>
        </thead>
        <tbody>
        <tr
            v-for="(request, index) in data"
            :key="index"
        >
          <td>{{ request.team2.name }}</td>
          <td>{{ request.tournament.name }}</td>

          <!--              <td>-->
          <!--                <v-btn-->
          <!--                    class="pa-0"-->
          <!--                    @click.stop="()=>{dialog_item = request.team;dialog = true;}"-->
          <!--                    text plain-->
          <!--                >-->
          <!--                  <v-icon-->
          <!--                  >mdi-account-box-outline-->
          <!--                  </v-icon>-->
          <!--                </v-btn>-->
          <!--              </td>-->
          <td class="text-center">
            <v-btn :disabled="!request.log" icon :href="request.log">
              <v-icon>mdi-download</v-icon>
            </v-btn>
          </td>
          <td class="text-center">
            <v-chip
                color="primary"
                v-if="request.status === 'pending'"
            >
              <v-icon class="ml-2">mdi-clock-time-four-outline</v-icon>
              در انتظار داوری
            </v-chip>
            <v-chip
                color="primary"
                v-else-if="request.status === 'running'"
            >
              <v-icon class="ml-2">mdi-clock-time-four-outline</v-icon>
              درحال اجرا
            </v-chip>
            <v-chip
                color="secondary"
                v-else-if="request.status === 'failed'"
            >
              <v-icon class="ml-2">mdi-close</v-icon>
              خطا
            </v-chip>
            <v-chip
                color="success"
                v-else-if="request.status === 'successful'"
            >
              <v-icon class="ml-2">mdi-check</v-icon>
              به اتمام رسیده
            </v-chip>


          </td>
          <td class="text-center">
            {{ !request.winner ? '-' : request.winner.name }}
          </td>
        </tr>
        </tbody>
      </template>
    </v-simple-table>
    <div v-if="data && data.length === 0" class="mb-10 px-md-12">
      لیست بازی های شما خالی است
    </div>
  </div>
</template>

<script>
import SectionHeader from '~/components/SectionHeader';
import SectionContainer from '~/components/SectionContainer';
import Logo from '~/components/dashboard/Logo';

export default {
  components: {SectionHeader, SectionContainer, Logo},
  async fetch() {
    let tournomentId = this.$route.query.id;
    this.url = tournomentId ? `challenge/match?tournament_id=${tournomentId}&` : 'challenge/match?';
    this.tableLoading = true;
    let filter = this.filterChip === 0 ? '&status=successful' : '';
    try {
      let res = await this.$axios.$get(`${this.url}page=${this.page}${filter}`);
      this.data = res.results;
      const count = 20;
      this.pageCount = Math.ceil(res.count / count);
      this.status_code = 200;

    } catch (e) {
      if (e.response) {
        if (e.response.status === 403) {
          this.$toast.error('برای مشاهده این صفحه باید تیم داشته باشید');
        } else {
          this.$toast.error('خطا در برقراری ارتباط!');
        }
      }
    }

    let team = await this.$axios.$get('team');
    this.myteam = team.name;
    this.tableLoading = false;
  },
  data() {
    return {
      filterSelect: 'همه',
      filterStatus: 'همه',
      filteIitems: ['همه', 'دوستانه', 'رقابتی', 'آزمایشی'],
      filterChip: null,
      tableLoading: false,
      btnLoading: false,
      dialog: false,
      page: 1,
      pageCount: 0,
      itemPerPage: 20,
      url: '',
      headers: [
        {
          text: 'بازی',
          align: 'right',
          sortable: false,
          value: 'x',
        },
        // { text: 'زمان', align: 'center', value: '' },
        {text: 'وضعیت بازی', align: 'center', value: 'status'},
        {text: 'تیم برنده', align: 'center', value: 'winner.name'},
        {text: 'لاگ', align: 'center', value: 'log'},
        {text: 'لاگ سرور', align: 'center', value: 'serverLog'},
        {text: 'پخش بازی', align: 'center', value: 'graphic'},
      ],
      data: [],
      currentGame: {
        serverLog: '',
        graphicLog: '',
      },
      status_code: 200,
      item: null,
      myteam: {},
    };
  },
  watch: {
    page() {
      this.$fetch();
    },
  },
  methods: {
    gameStatus(status) {
      switch (status) {
        case 'freeze':
          return 'ثبت شده';
        case 'pending':
          return 'در صف اجرا';
        case 'running':
          return 'در حال اجرا';
        case 'failed':
          return 'اجرا با خطا';
        case 'successful':
          return 'تمام ‌شده';
      }
    },
    filter(data) {
      this.tableLoading = true;
      this.btnLoading = true;

      var lastApi = '********************';

      if (data === 'دوستانه') {
        lastApi = lastApi + '?' + '******************' + '=' + '******************';
      } else if (data === 'رقابتی') {
        lastApi = lastApi + '?' + '******************' + '=' + '******************';
      } else if (data === 'آزمایشی') {
        lastApi = lastApi + '?' + '******************' + '=' + '******************';
      }

      this.$axios.$get(lastApi).then(res => {
        if (res.status_code === 200) {
          this.data = res.data;
          this.status_code = res.status_code;
        } else {
          this.$toast.error('خطا در برقراری ارتباط!');
        }
      });
      this.btnLoading = false;
      this.tableLoading = false;
    },
    handleFilterChip() {
      this.$fetch();
    },
    resultTrasnlate(res) {
      if (res === '') return 'برد';
      else if (res === '') return 'باخت';
      else if (res === '') return 'مساوی';
      else if (res === '') return 'در حال اجرا';
    },
    resultIcon(res) {
      if (res === '') return 'mdi-emoticon-cool-outline';
      else if (res === '') return 'mdi-emoticon-cry-outline';
      else if (res === '') return 'mdi-emoticon-happy-outline';
      else if (res === '') return 'mdi-progress-clock';
    },
  },
};
</script>

<style scoped lang="scss">
.myteam {
  color: var(--v-success-lighten1);
}

.header {
  display: flex;
  justify-content: space-between;
}

.select-filter {
  display: flex;
}

.dialog-header {
  display: flex;
  justify-content: space-between;
}

.profile {
  display: flex;
  align-items: center;
}

.icon-hover {
  &:hover {
    color: var(--v-primary-base);
  }
}
</style>
