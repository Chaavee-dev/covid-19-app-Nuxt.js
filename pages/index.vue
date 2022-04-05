<style scoped>
.select {
  width: 300px;
}
.text-field {
  width: 300px;
}
</style>

<template>
  <v-row align="center" justify="center">
    <v-col class="text-center" cols="12" lg="12" sm="6">
      <img
        src="/images/kingdom-704.png"
        alt="Vuetify.js"
        class="mb-5"
        width="100px"
      />
      <blockquote class="blockquote">
        &#8220;สถานการณ์ผู้ติดเชื้อ COVID-19 อัพเดทรายวัน.&#8221;
        <footer>
          <small>
            <em>&mdash;Covid-19 | TH</em>
          </small>
        </footer>
      </blockquote>
    </v-col>
    <v-col class="text-center" cols="12" lg="12" sm="6">
      <p v-if="data.txn_date">
        โควิด{{ title }} :
        {{
          new Date(data.txn_date).toLocaleDateString('th-TH', {
            year: 'numeric',
            month: 'long',
            day: 'numeric',
            weekday: 'long',
          })
        }}
      </p>
      <p v-else>
          กำลังโหลด...
      </p>
    </v-col>
    <v-col align="center" cols="12" lg="6" sm="6">
      <v-select
        class="select"
        :items="provinces"
        label="เลือกจังหวัดที่คุณต้องการตรวจสอบ"
        outlined
        dense
        v-model="selected"
        :loading="provinces.length == 1"
        :disabled="provinces.length == 1"
        @change="onChange()"
      ></v-select>
    </v-col>
    <v-col align="center" cols="12" lg="6" sm="6">
      <v-dialog
        ref="dialog"
        v-model="modal"
        :return-value.sync="date"
        persistent
        width="290px"
      >
        <template v-slot:activator="{ on, attrs }">
          <v-text-field
            class="text-field"
            v-model="date"
            label="เลือก วัน/เดือน/ปี ที่คุณต้องการตรวจสอบ"
            readonly
            v-bind="attrs"
            v-on="on"
            outlined
            dense
          ></v-text-field>
        </template>
        <v-date-picker v-model="date" min="2020-01-12" :max="(new Date()).toISOString().substr(0, 10)" scrollable>
          <v-spacer></v-spacer>
          <v-btn text color="red" @click="modal=false"> ยกเลิก </v-btn>
          <v-btn
            text
            color="success"
            @click="
              $refs.dialog.save(date)
              fetchDATA_Date()
            "
          >
            ตกลง
          </v-btn>
        </v-date-picker>
      </v-dialog>
    </v-col>
    <v-col class="d-flex" cols="12" lg="3" sm="6">
      <v-card
        class="mx-auto"
        :disabled="!data"
        :loading="!data"
        color="#dc5968"
        dark
        width="100%"
      >
        <div class="d-flex flex-no-wrap justify-center">
          <v-avatar class="ma-3" size="125">
            <v-img
              src="/images/istockphoto-1356114754-170667a.jpg"
              height="200px"
            ></v-img>
          </v-avatar>
          <div>
            <v-card-title class="text-h6">ผู้ติดเชื้อรายใหม่</v-card-title>
            <v-card-text v-if="data.new_case >= 0" class="text-h4"
              >{{ Number(data.new_case).toLocaleString() }} คน</v-card-text
            >
            <v-card-text v-else class="text-h5"
              >กำลังโหลด...</v-card-text
            >
          </div>
        </div>
      </v-card>
    </v-col>

    <v-col class="d-flex" cols="12" lg="3" sm="6">
      <v-card
        class="mx-auto"
        :disabled="!data"
        :loading="!data"
        color="#7f7f7f"
        dark
        width="100%"
      >
        <div class="d-flex flex-no-wrap justify-center">
          <v-avatar class="ma-3" size="125">
            <v-img src="/images/covid.jpg" height="200px"></v-img>
          </v-avatar>
          <div>
            <v-card-title class="text-h6">ผู้เสียชีวิตรายใหม่</v-card-title>
            <v-card-text v-if="data.new_death >= 0" class="text-h4"
              >{{ Number(data.new_death).toLocaleString() }} คน</v-card-text
            >
            <v-card-text v-else class="text-h5"
              >กำลังโหลด...</v-card-text
            >
          </div>
        </div>
      </v-card>
    </v-col>

    <v-col class="d-flex" cols="12" lg="3" sm="6" v-if="data.new_recovered">
      <v-card
        class="mx-auto"
        :disabled="!data"
        :loading="!data"
        color="#a0c7f0"
        dark
        width="100%"
      >
        <div class="d-flex flex-no-wrap justify-center">
          <v-avatar class="ma-3" size="125">
            <v-img
              src="/images/istockphoto-1308624310-170667a.jpg"
              height="200px"
            ></v-img>
          </v-avatar>
          <div>
            <v-card-title class="text-h6">กำลังรักษา</v-card-title>
            <v-card-text v-if="data.new_recovered >= 0" class="text-h4"
              >{{ Number(data.new_recovered).toLocaleString() }} คน</v-card-text
            >
            <v-card-text v-else class="text-h5"
              >กำลังโหลด...</v-card-text
            >
          </div>
        </div>
      </v-card>
    </v-col>
    <v-col class="d-flex" cols="12" lg="3" sm="6">
      <v-card
        class="mx-auto"
        :disabled="!data"
        :loading="!data"
        color="#97d256"
        dark
        width="100%"
      >
        <div class="d-flex flex-no-wrap justify-center">
          <v-avatar class="ma-3" size="125">
            <v-img src="/images/Coronavirus_pillars.jpg" height="200px"></v-img>
          </v-avatar>
          <div>
            <v-card-title class="text-h6">รักษาหาย</v-card-title>
            <v-card-text v-if="data.new_case_excludeabroad >= 0" class="text-h4"
              >{{
                Number(data.new_case_excludeabroad).toLocaleString()
              }}
              คน</v-card-text
            >
            <v-card-text v-else class="text-h5"
              >กำลังโหลด...</v-card-text
            >
          </div>
        </div>
      </v-card>
    </v-col>
    <v-col class="text-center" cols="12" lg="12" sm="6">
      <a href="https://covid19.ddc.moph.go.th/" target="_blank">
        <img src="/images/sat-moph.jpg" alt="" />
      </a>
    </v-col>
  </v-row>
</template>

<script>

export default {
  name: 'HomePage',
  data: () => ({
    data: false,
    modal: false,
    title: 'ประเทศไทย',
    provinces: ['ทั้งหมด'],
    selected: 'ทั้งหมด',
    date: new Date(Date.now() - new Date().getTimezoneOffset() * 60000)
      .toISOString()
      .substr(0, 10),
  }),
  mounted() {
    this.fetchDATA()
    this.fetchProvinces()
  },
  methods: {
    onChange() {
      this.data = false
      // this.$router.push({path: '?province='+this.selected })
      // console.log(this.selected);
      if (this.selected == 'ทั้งหมด') {
        this.title = 'ประเทศไทย'
        document.title = 'COVID-19 | ประเทศไทย'
        if(this.date == (new Date()).toISOString().substr(0, 10)){
            this.fetchDATA()
        }else{
            this.fetchDATA_Date()
        }
      } else {
          if(this.date == (new Date()).toISOString().substr(0, 10)){
            this.fetchDATA_Province()
        }else{
            this.fetchDATA_Date()
        }
      }
    },

    async fetchDATA() {
      const url = 'https://covid19.ddc.moph.go.th/api/Cases/today-cases-all'
      const data = await this.$axios.$get(url)
      this.data = data[0]
      //   console.log(data[0])
      //   console.log(navigator.geolocation);
    },
    async fetchDATA_Date() {
      this.data = false
      let url
      let data
      let dataFilter
      const d = new Date(this.date)
      if ((d.getMonth() >= 3 && d.getFullYear() == 2021) || (d.getFullYear() >= 2022)) {
        if (this.selected == 'ทั้งหมด') {
          url = 'https://covid19.ddc.moph.go.th/api/Cases/timeline-cases-all'
          data = await this.$axios.$get(url)
          dataFilter = await data.filter((e) => e.txn_date == this.date)
        } else {
          url =
            'https://covid19.ddc.moph.go.th/api/Cases/timeline-cases-by-provinces'
          data = await this.$axios.$get(url)
          dataFilter = await data.filter(
            (e) => e.province == this.selected && e.txn_date == this.date
          )
        }
        this.data = dataFilter[0]
      } else if ((d.getMonth() <= 2 && d.getFullYear() == 2021) || (d.getFullYear() == 2020)) {
        if (this.selected == 'ทั้งหมด') {
          url = 'https://covid19.ddc.moph.go.th/api/Cases/round-1to2-all'
          data = await this.$axios.$get(url)
          dataFilter = await data.filter((e) => e.txn_date == this.date)
        } else {
          url =
            'https://covid19.ddc.moph.go.th/api/Cases/round-1to2-by-provinces'
          data = await this.$axios.$get(url)
          dataFilter = await data.filter(
            (e) => e.province == this.selected && e.txn_date == this.date
          )
        }
        this.data = dataFilter[0]
      } else {
        console.log('ERROR!!!')
      }
      //   this.data = dataFilter[0]
      //   console.log(this.data);
    },
    async fetchProvinces() {
      const url =
        'https://covid19.ddc.moph.go.th/api/Cases/today-cases-by-provinces'
      const provinces = await this.$axios.$get(url)
      //   this.provinces = provinces
      //   console.log(provinces)
      provinces.forEach((e) => this.provinces.push(e.province))
      this.provinces.pop('ไม่ระบุ')
    },
    async fetchDATA_Province() {
      this.data = false
      const current = new Date()
      const d = new Date(this.date)
      if (
        current.getMonth() == d.getMonth() &&
        current.getFullYear() == d.getFullYear()
      ) {
        const url =
          'https://covid19.ddc.moph.go.th/api/Cases/today-cases-by-provinces'
        const data = await this.$axios.$get(url)
        const dataFilter = await data.filter((e) => e.province == this.selected)
        this.data = await dataFilter[0]
      } else {
        this.fetchDATA_Date()
      }
      this.title = 'จังหวัด' + this.selected
      document.title = 'COVID-19 | จังหวัด' + this.selected

      //   console.log(data[0])
    },
  },
}
</script>
