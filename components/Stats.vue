<template>
  <div class="MainPage">
    <v-row class="DataBlock">
      <!-- Summary of Bay Area Counties Stats Card -->
      <v-col cols="12" md="12" class="DataCard">
        <cases-summary
          :title="'Summary for 9 Bay Area Counties'"
          :title-id="'confirmed-cases'"
          :data="ConsolidatedData"
          :date="CountyData[currentCounty].lastUpdatedAt"
          :url="'https://coronadatascraper.com'"
        />
      </v-col>
      <!-- Bay Area Graphs -->
      <v-col cols="12" md="6" class="DataCard">
        <time-bar-chart
          :title="`Confirmed Cases: Bay Area Total`"
          :title-id="'number-of-confirmed-cases'"
          :chart-id="'time-bar-chart-patients'"
          :chart-data="ConsolidatedData.cases"
          :chart-data-type="'cases'"
          :date="CountyData[currentCounty].lastUpdatedAt"
          :url="'https://coronadatascraper.com'"
        />
      </v-col>
      <v-col cols="12" md="6" class="DataCard">
        <time-bar-chart
          :title="`COVID Related Deaths: Bay Area Total`"
          :title-id="'number-of-confirmed-cases'"
          :chart-id="'time-bar-chart-patients'"
          :chart-data="ConsolidatedData.cases"
          :chart-data-type="'deaths'"
          :date="CountyData[currentCounty].lastUpdatedAt"
          :url="'https://coronadatascraper.com'"
        />
      </v-col>
      <!-- County Selector & Stats Card -->
      <v-col cols="12" md="12" class="DataCard">
        <DataView>
          <div class="county-select-container">
            <div class="county-select">
              <label class="selection">Show Data For:</label>
              <select v-model="currentCounty" class="select-css">
                <option v-for="(countyName, index) in countyNames" :key="index">
                  {{ countyName }}
                </option>
              </select>
            </div>
            <div class="county-stats">
              <div class="border">
                <span class="stat-title">Confirmed Cases</span>
                <div class="stat-number" :county="Data[currentCounty]">
                  {{ getCurrentCountyLatestCases.cases.toLocaleString() }}
                </div>
                <div class="stat-note">
                  <strong>{{
                    `${(
                      (getCurrentCountyLatestCases.cases / totalCases) *
                      100
                    ).toFixed(2)}% `
                  }}</strong>
                  of Bay Area Total
                </div>
              </div>
              <div>
                <span class="stat-title">Deaths</span>
                <div class="stat-number">
                  {{ getCurrentCountyLatestCases.deaths.toLocaleString() }}
                </div>
                <div class="stat-note">
                  <strong>{{
                    `${(
                      (getCurrentCountyLatestCases.deaths / totalDeaths) *
                      100
                    ).toFixed(2)}% `
                  }}</strong>
                  of Bay Area Total
                </div>
              </div>
            </div>
          </div>
        </DataView>
      </v-col>
      <!-- Selected County Graphs -->
      <v-col
        :county="CountyData[currentCounty]"
        cols="12"
        md="6"
        class="DataCard"
      >
        <time-bar-chart
          :title="`Confirmed Cases: ${CountyData[currentCounty].name}`"
          :title-id="'number-of-confirmed-cases'"
          :chart-id="'time-bar-chart-patients'"
          :chart-data="CountyData[currentCounty].graph"
          :chart-data-type="'cases'"
          :date="CountyData[currentCounty].lastUpdatedAt"
          :url="'https://coronadatascraper.com'"
        />
      </v-col>
      <v-col
        :county="CountyData[currentCounty]"
        cols="12"
        md="6"
        class="DataCard"
      >
        <time-bar-chart
          :title="`COVID Related Deaths: ${CountyData[currentCounty].name}`"
          :title-id="'number-of-deaths'"
          :chart-id="'time-bar-chart-patients'"
          :chart-data="CountyData[currentCounty].graph"
          :chart-data-type="'deaths'"
          :date="CountyData[currentCounty].lastUpdatedAt"
          :url="'https://coronadatascraper.com'"
        />
      </v-col>
      <v-col cols="12" md="6" class="DataCard">
        <horizontal-bar-chart
          :title="`County Cases by Age: ${CountyData[currentCounty].name}`"
          :title-id="'cases-by-age'"
          :chart-id="'horizontal-bar-chart-age'"
          :chart-data="CountyDataVTwo[currentCounty].ageGroup"
          :date="CountyDataVTwo[currentCounty].lastUpdatedAt"
          :url="CountyDataVTwo[currentCounty].sourceUrl"
        />
      </v-col>
      <v-col cols="12" md="6" class="DataCard">
        <horizontal-bar-chart
          :title="`Confirmed Cases by Sex: ${CountyData[currentCounty].name}`"
          :title-id="'cases-by-gender'"
          :chart-id="'horizontal-bar-chart-gender'"
          :chart-data="CountyDataVTwo[currentCounty].genderGroup"
          :date="CountyDataVTwo[currentCounty].lastUpdatedAt"
          :url="CountyDataVTwo[currentCounty].sourceUrl"
        />
      </v-col>
      <v-col cols="12" md="6" class="DataCard">
        <horizontal-bar-chart
          :title="
            `County Cases by Race and Ethnicity: ${CountyData[currentCounty].name}`
          "
          :title-id="'cases-by-race-eth'"
          :chart-id="'horizontal-bar-chart-race-eth'"
          :chart-data="CountyDataVTwo[currentCounty].raceEthGroup"
          :chart-info="chartInfo.raceEth"
          :date="CountyDataVTwo[currentCounty].lastUpdatedAt"
          :url="CountyDataVTwo[currentCounty].sourceUrl"
        />
      </v-col>
      <v-col cols="12" md="6" class="DataCard">
        <horizontal-bar-chart
          :title="
            `County Cases per 1000 by Race and Ethnicity: ${CountyData[currentCounty].name}`
          "
          :title-id="'cases-by-race-eth-norm'"
          :chart-id="'horizontal-bar-chart-race-eth-norm'"
          :chart-data="CountyDataVTwo[currentCounty].raceEthNormGroup"
          :chart-info="chartInfo.raceEthNorm"
          :date="CountyDataVTwo[currentCounty].lastUpdatedAt"
          :url="CountyDataVTwo[currentCounty].sourceUrl"
        />
      </v-col>
      <v-col cols="12" md="6" class="DataCard">
        <time-line-chart
          :title="
            `COVID ICU Care and Capacity: ${CountyData[currentCounty].name}`
          "
          :title-id="'icu-capacity'"
          :chart-data="
            CountyDataHospitalization[
              renameCountyNameForHospitalization(currentCounty)
            ].graph
          "
          :date="
            CountyDataHospitalization[
              renameCountyNameForHospitalization(currentCounty)
            ].lastUpdatedAt
          "
          :url="
            CountyDataHospitalization[
              renameCountyNameForHospitalization(currentCounty)
            ].sourceUrl
          "
        />
      </v-col>
      <!-- County Comparison Selector -->
      <v-col cols="12" md="12" class="DataCard">
        <DataView>
          <div class="county-compare-select-container">
            <label>Select Counties to Compare:</label>
          </div>
          <div class="county-select-buttons">
            <v-btn
              v-for="countyName in countiesForCompare"
              :key="countyName.name"
              class="county-select-button"
              outlined
              :style="{
                'background-color': contains(selectedCounties, countyName)
                  ? countyName.color
                  : 'white',
                color: contains(selectedCounties, countyName)
                  ? 'white'
                  : 'black'
              }"
              type="button"
              @click="provide(countyName)"
            >
              {{ countyName.name }}
            </v-btn>
          </div>
        </DataView>
      </v-col>
      <!-- County Comparison Graphs -->
      <v-col
        :county="CountyData[currentCounty]"
        cols="12"
        md="6"
        class="DataCard"
      >
        <time-line-chart-county-comparison
          :title="`New Cases per 100,000 Residents`"
          :sub-title="'14 day rolling average'"
          :title-id="'cases-per-people'"
          :chart-data="CountyData"
          :selected-counties="selectedCounties"
          :chart-data-type="'casesperpeople'"
          :chart-info="chartInfo.casesPerResidents"
          :date="CountyData[currentCounty].lastUpdatedAt"
          :url="'https://coronadatascraper.com'"
        />
      </v-col>
      <v-col
        :county="CountyData[currentCounty]"
        cols="12"
        md="6"
        class="DataCard"
      >
        <time-line-chart-county-comparison
          :title="`Percent Increase in 7 Days`"
          :title-id="'percent-increase-in7days'"
          :chart-data="CountyData"
          :selected-counties="selectedCounties"
          :chart-data-type="'percentincrease'"
          :date="CountyData[currentCounty].lastUpdatedAt"
          :unit="'%'"
          :url="'https://coronadatascraper.com'"
        />
      </v-col>
    </v-row>
  </div>
</template>

<script>
import TimeBarChart from '@/components/TimeBarChart.vue'
import TimeLineChart from '@/components/TimeLineChart.vue'
import TimeLineChartCountyComparison from '@/components/TimeLineChartCountyComparison.vue'
import CasesSummary from '@/components/CasesSummary.vue'
import HorizontalBarChart from '@/components/HorizontalBarChart'
import Data from '@/data/data.json'
import DataVTwo from '@/data/data.v2.json'
import DataHospitalization from '@/data/data_hospitalization.json'
import formatCountyData from '@/utils/formatCountyData'
import formatCountyDataVTwo from '@/utils/formatCountyDataVTwo'
import consolidateAllData from '@/utils/consolidateAllData'
import formatCountyHospitalizationData from '@/utils/formatCountyHospitalizationData'
import DataView from '@/components/DataView.vue'
import countyColor from '@/static/data/countyColor.json'

export default {
  components: {
    CasesSummary,
    TimeBarChart,
    TimeLineChart,
    TimeLineChartCountyComparison,
    HorizontalBarChart,
    DataView
  },
  data() {
    const currentCounty = 'San Francisco County'
    const CountyData = formatCountyData(Data)
    const ConsolidatedData = consolidateAllData(Data)
    const countyNames = Object.keys(Data)

    const totalCases =
      ConsolidatedData.cases[ConsolidatedData.cases.length - 1].cumulative
    const totalDeaths =
      ConsolidatedData.cases[ConsolidatedData.cases.length - 1].deathCumulative

    const countiesForCompare = []
    for (const countyName of countyNames) {
      countiesForCompare.push({
        name: countyName,
        color: countyColor[countyName]
      })
    }
    const selectedCounties = []
    const CountyDataVTwo = this.getFormatData()
    const chartInfo = this.getChartInfo()

    const CountyDataHospitalization = formatCountyHospitalizationData(
      DataHospitalization
    )

    const data = {
      Data,
      DataVTwo,
      DataHospitalization,
      CountyData,
      CountyDataVTwo,
      CountyDataHospitalization,
      ConsolidatedData,
      currentCounty,
      countyNames,
      totalCases,
      totalDeaths,
      countiesForCompare,
      selectedCounties,
      chartInfo
    }

    return data
  },
  computed: {
    getCurrentCountyLatestCases() {
      let offsetDay = 0
      let cases = 0
      let deaths = 0
      // Sometimes data does not contain proper number so we need to find the latest valid number
      while (
        cases === undefined ||
        cases === 0 ||
        deaths === undefined ||
        deaths === 0
      ) {
        offsetDay++
        cases =
          Data[this.currentCounty].cases[
            Data[this.currentCounty].cases.length - offsetDay
          ].cases
        deaths =
          Data[this.currentCounty].cases[
            Data[this.currentCounty].cases.length - offsetDay
          ].deaths
      }

      return {
        cases,
        deaths
      }
    }
  },
  methods: {
    provide(item) {
      if (!this.selectedCounties.includes(item)) {
        this.selectedCounties.push(item)
      } else {
        this.selectedCounties.splice(this.selectedCounties.indexOf(item), 1)
      }
    },
    contains(arr, item) {
      return arr.includes(item)
    },
    getFormatData() {
      const allCounties = Object.keys(Data)
      return formatCountyDataVTwo(DataVTwo, allCounties)
    },
    getChartInfo() {
      return {
        raceEth: [
          {
            title: 'What this graph shows',
            description:
              'This chart shows the racial/ethnic breakdown of county cases. This information is gathered from multiple sources including medical records, testing labs and interviews. The large number of unknown cases should be noted when assessing the significance of this data.'
          }
        ],
        raceEthNorm: [
          {
            title: 'What this graph shows',
            description:
              'Showing cases per 1000 people normalizes the data for population size. This measure reflects the relative prevelance of COVID-19 cases by race and ethnicity. This data is calculated based on US census population data and confirmed cases that registered racial/ethnic data.'
          }
        ],
        casesPerResidents: [
          {
            title: 'What this graph shows',
            description:
              'The daily values shown on this graph are the 14-day rolling average for new confirmed cases adjusted for county population size.'
          },
          {
            title: 'Why ‘per 100,000 residents’?',
            description:
              'Showing a population-adjusted value puts larger and smaller counties on the same scale and allows for more accurate comparison.'
          },
          {
            title: 'Why a 14 day average?',
            description:
              'Showing an average value smooths the data curve and makes trends easier to observe. The 14 day period was chosen to match the maximum time frame between contraction of the virus and the onset of symptoms.'
          }
        ]
      }
    },
    renameCountyNameForHospitalization(name) {
      switch (name) {
        case 'Alameda County':
          return 'alameda'
        case 'Contra Costa County':
          return 'contra_costa'
        case 'Marin County':
          return 'marin'
        case 'Napa County':
          return 'napa'
        case 'San Francisco County':
          return 'san_francisco'
        case 'San Mateo County':
          return 'san_mateo'
        case 'Santa Clara County':
          return 'santa_clara'
        case 'Sonoma County':
          return 'sonoma'
        case 'Solano County':
          return 'solano'
      }
    }
  },
  head() {
    return {
      title: 'Bay Area Pandemic Dashboard'
    }
  }
}
</script>

<style lang="scss" scoped>
.MainPage {
  .DataBlock {
    margin: 20px -8px;
    .DataCard {
      position: relative;
      @include largerThan($medium) {
        padding: 10px;
      }
      @include lessThan($small) {
        padding: 4px 8px;
      }
    }
    .county-select-container {
      display: grid;
      grid-template-columns: repeat(2, 1fr);
      align-items: center;
      label {
        font-style: normal;
        font-weight: bold;
        font-size: 24px;
        color: black;
        line-height: 33px;
        margin-right: 10px;
      }
      .county-stats {
        display: grid;
        grid-template-columns: repeat(2, 1fr);
        text-align: center;
        .stat-title {
          font-size: 16px;
          color: black;
          font-weight: bold;
          line-height: 22px;
        }
        .stat-number {
          padding: 10px;
          margin: 20px 0px;
          color: black;
          font-weight: bold;
        }
        .stat-note {
          font-style: normal;
          font-weight: 500;
          font-size: 12px;
          line-height: 15px;
          color: $gray-1;
        }
        .border {
          border-right: 2px solid lightgray;
        }
      }
      .county-select {
        display: flex;
        justify-content: center;
        align-items: center;
      }
    }
    .county-compare-select-container {
      display: grid;
      align-items: center;
      label {
        text-align: center;
        font-style: normal;
        font-weight: bold;
        font-size: 24px;
        color: black;
        line-height: 33px;
      }
    }
    .county-select-buttons {
      padding: 20px;
    }
    .county-select-button {
      margin: 10px 10px;
    }
    @include lessThan($small) {
      .county-select-container {
        display: initial;
        grid-template-columns: initial;
        .county-stats {
          display: initial;
          grid-template-columns: initial;
          .stat-title {
            margin-top: 50px;
          }
          .stat-number {
            padding: 10px;
            margin: 20px 0px;
            font-size: 36px;
          }
          .stat-note {
          }
          .border {
            border-right: initial;
            margin-bottom: 30px;
          }
        }
        .county-select {
          display: grid;
        }
        .selection {
          text-align: center;
        }
      }
    }
  }

  .select-css {
    display: block;
    font-size: 16px;
    font-weight: 700;
    color: #444;
    line-height: 1.3;
    padding: 0.6em 1.4em 0.5em 0.8em;
    width: 40%;
    max-width: 60%;
    box-sizing: border-box;
    margin: 0;
    border: 1px solid #aaa;
    box-shadow: 0 1px 0 1px rgba(0, 0, 0, 0.04);
    border-radius: 0.5em;
    -moz-appearance: none;
    -webkit-appearance: none;
    appearance: none;
    background-color: #fff;
    background-image: url('data:image/svg+xml;charset=US-ASCII,%3Csvg%20xmlns%3D%22http%3A%2F%2Fwww.w3.org%2F2000%2Fsvg%22%20width%3D%22292.4%22%20height%3D%22292.4%22%3E%3Cpath%20fill%3D%22%23007CB2%22%20d%3D%22M287%2069.4a17.6%2017.6%200%200%200-13-5.4H18.4c-5%200-9.3%201.8-12.9%205.4A17.6%2017.6%200%200%200%200%2082.2c0%205%201.8%209.3%205.4%2012.9l128%20127.9c3.6%203.6%207.8%205.4%2012.8%205.4s9.2-1.8%2012.8-5.4L287%2095c3.5-3.5%205.4-7.8%205.4-12.8%200-5-1.9-9.2-5.5-12.8z%22%2F%3E%3C%2Fsvg%3E'),
      linear-gradient(to bottom, #ffffff 0%, #e5e5e5 100%);
    background-repeat: no-repeat, repeat;
    background-position: right 0.7em top 50%, 0 0;
    background-size: 0.65em auto, 100%;
    &:-ms-expand {
      display: none;
    }
    &:hover {
      border-color: #888;
    }
    &:focus {
      border-color: #aaa;
      box-shadow: 0 0 1px 3px rgba(59, 153, 252, 0.7);
      box-shadow: 0 0 0 3px -moz-mac-focusring;
      color: #222;
      outline: none;
    }
    option {
      font-weight: normal;
    }
  }
  @include lessThan($small) {
    .select-css {
      width: initial;
      max-width: initial;
      margin-top: 20px;
      margin-bottom: 40px;
      align-items: center;
    }
  }
}
@media screen and (min-width: 640px) {
  .stat-number {
    font-size: 36px;
  }
}
@media screen and (max-width: 640px) {
  .stat-number {
    font-size: 30px;
  }
  h4 {
    font-size: 15px;
  }
  label {
    font-size: 30px;
  }
}
</style>
