<template>
  <div class="columns is-centered">
    <div class="column is-two-thirds">
      <article>
        <div class="dx">
          <h1
            class="is-size-3-desktop is-size-3-mobile is-bold mb-5 has-text-left"
          >
            DataCabin 🏡
          </h1>
          <p class="has-text-left">
            Perform fast lightweight automated exploratory data analysis on your CSV datasets.
            Watch the <a
                  href="https://www.youtube.com/watch?v=fev2rpGxh-U"
                  rel="nofollow noopener noreferrer"
                  target="_blank"
                  >tutorial video</a>.
                  <b-tooltip 
                    label="No data from your CSV is sent to a server or anywhere else. All
                      the data stays on your computer and the calculations are made
                      within your browser using JavaScript. Datasets over 20,000 rows
                      will slow down the tool considerably." 
                    multilined 
                    dashed
                    type="is-light">
                      Privacy focused.
                  </b-tooltip>
          </p>
          <div class="columns">
            <div id="main-column" class="column is-full">
              <b-field v-if="dropFiles.length == 0">
                <b-upload v-model="dropFiles" accept=".csv" drag-drop>
                  <section class="section">
                    <div class="content has-text-centered">
                      <p v-if="isLoading">
                        <svg
                          xmlns="http://www.w3.org/2000/svg"
                          width="75"
                          height="75"
                          stroke="#000"
                          viewBox="0 0 24 24"
                        >
                          <g class="spinner_V8m1">
                            <circle
                              cx="12"
                              cy="12"
                              r="9.5"
                              fill="none"
                              stroke-width="3"
                            />
                          </g>
                        </svg>
                      </p>
                      <p>
                        <svg
                          xmlns="http://www.w3.org/2000/svg"
                          width="100"
                          height="100"
                          viewBox="0 0 24 24"
                        >
                          <path
                            fill="currentColor"
                            d="M9 16v-6H5l7-7l7 7h-4v6H9m-4 4v-2h14v2H5Z"
                          />
                        </svg>
                      </p>
                      <p>Drop your CSV dataset here or click to upload</p>
                    </div>
                  </section>
                </b-upload>
              </b-field>
              <b-button 
                v-if="dropFiles.length == 0"
                type="is-text" 
                @click="fetchTitanicData()">
                To get started even quicker, you can auto-load the Titanic dataset
              </b-button>.
              <p class="is-size-6" v-if="dropFiles.length == 0">
                ... or find some 
                <a href="https://github.com/MainakRepositor/Datasets/tree/master" 
                  target="_blank"
                  rel="nofollow noopener noreferrer"
                  style="font-weight: 100"
                  >public datasets</a> to use
              </p>
              <div class="tags is-centered" v-if="dropFiles.length != 0">
                <span class="tag is-small is-secondary">
                  {{ dropFiles.name }}
                  <button
                    class="delete is-small"
                    type="button"
                    @click="deleteDropFile(dropFiles.index)"
                  ></button>
                </span>
                <span class="tag is-small is-secondary">
                  {{ dataInfo.numberOfRows }} rows
                </span>
                <span class="tag is-small is-secondary">
                  {{ dataInfo.numberOfColumns }} columns
                </span>
                <span class="tag is-small is-secondary">
                  {{ dataInfo["missingValues%"] }}% missing
                </span>
              </div>
              <b-tabs
                :animated="false"
                size="is-small"
                class="block"
                position="is-centered"
                v-if="dropFiles.length != 0"
                v-model="activeTab"
                @input="tabClicked(activeTab)"
              >
                <b-tab-item label="Summary">
                  <div class="columns">
                    <div class="column is-half">
                      <table class="table" v-if="this.df != null && !isLoading">
                        <thead class="dx-thead">
                          Dataset statistics
                        </thead>
                        <tbody>
                          <tr>
                            <th width="100%">Number of variables (columns)</th>
                            <td width="100%">
                              {{ this.df.shape[1] }}
                            </td>
                          </tr>
                          <tr>
                            <th>Number of observations (rows)</th>
                            <td>
                              {{ this.df.shape[0] }}
                            </td>
                          </tr>
                          <tr>
                            <th>Missing values</th>
                            <td>
                              {{ dataInfo.totalMissingValues }}
                            </td>
                          </tr>
                          <tr>
                            <th>Missing values %</th>
                            <td>
                              {{ dataInfo["missingValues%"] }}
                            </td>
                          </tr>
                          <tr>
                            <th>Duplicate rows</th>
                            <td>
                              {{ dataInfo.duplicateRows }}
                            </td>
                          </tr>
                          <tr>
                            <th>Duplicate rows %</th>
                            <td>
                              {{ dataInfo["duplicateRows%"] }}
                            </td>
                          </tr>
                          <tr>
                            <th>Numeric columns</th>
                            <td>
                              {{ dataInfo["numColumns"].$columns.length }}
                            </td>
                          </tr>
                          <tr>
                            <th>Categoric columns</th>
                            <td>
                              {{ dataInfo["catColumns"].$columns.length }}
                            </td>
                          </tr>
                        </tbody>
                      </table>
                    </div>
                    <div class="column is-half">
                      <client-only>
                        <table
                          class="table"
                          width="100%"
                          v-show="!isLoading && df != null"
                        >
                          <thead class="dx-thead">
                            Dataset information
                          </thead>
                          <tbody width="100%">
                            <tr
                              v-for="warning in dataInfo.warnings"
                              width="100%"
                              :key="warning.id"
                            >
                              <td width="100%" v-html="warning"></td>
                            </tr>
                            <!-- <tr>
                                <td>
                                    Survived has 549 (61.62%) zeros
                                </td>
                            </tr> -->
                          </tbody>
                        </table>
                      </client-only>
                    </div>
                  </div>
                </b-tab-item>
                <div style="overflow-x: auto; width: 100%;">
                  <b-tab-item label="Sample">
                    <client-only>
                      <table v-if="dataInfo.columns.length > 0" class="table">
                        <thead>
                          <tr>
                            <th
                              v-for="column in dataInfo.columns"
                              :key="column.id"
                            >
                              {{ column }}
                            </th>
                          </tr>
                        </thead>
                        <tbody>
                          <tr
                            v-for="row in df.head(20).fillNa('').$data"
                            :key="row.id"
                          >
                            <td
                              v-for="(value, i) in row"
                              v-bind:key="'rowValue' + i"
                            >
                              {{ value }}
                            </td>
                          </tr>
                        </tbody>
                      </table>
                    </client-only>
                  </b-tab-item>
                </div>
                <b-tab-item label="Variables">
                  <div v-show="variablePlotsInitialised">
                    <div
                      class="columns"
                      v-for="column in chunk(dataInfo.columns, 2)"
                      :key="column.index"
                    >
                      <div class="column is-half">
                        <div class="card">
                          <div class="card-image">
                            <div
                              :id="'plot_' + column[0].toLowerCase()"
                              v-if="!highCardinalityColumns.includes(column[0])"
                            >
                              <!-- Rendered plot goes here -->
                            </div>
                            <div v-else>
                              <figure class="image">
                                <img
                                  style="
                                    position: relative;
                                    height: 450px;
                                    width: 100%;
                                  "
                                  src="https://res.cloudinary.com/dayqxxsip/image/upload/v1690498677/too-many-unique-values-warning_ggrjws.png"
                                  alt="Placeholder image"
                                />
                              </figure>
                            </div>
                          </div>
                          <div class="card-content dx-card-content">
                            <div class="media">
                              <div class="media-left">
                                <figure class="image is-48x48">
                                  <img
                                    v-if="
                                      dataInfo.numColumns.$columns.includes(
                                        column[0]
                                      )
                                    "
                                    src="https://res.cloudinary.com/dayqxxsip/image/upload/v1601923068/Shared/1200px-Greek_uc_sigma.svg_hsngam.png"
                                  />
                                  <img
                                    v-else
                                    src="https://res.cloudinary.com/dayqxxsip/image/upload/v1601923065/Shared/category-icon-png-2_jhizjk.png"
                                  />
                                </figure>
                              </div>
                              <div class="media-content">
                                <p class="title is-4">{{ column[0] }}</p>
                                <p
                                  v-if="
                                    dataInfo.numColumns.$columns.includes(
                                      column[0]
                                    )
                                  "
                                  class="subtitle is-6"
                                >
                                  Numeric variable
                                </p>
                                <p v-else class="subtitle is-6">
                                  Categoric variable
                                </p>
                              </div>
                            </div>

                            <div
                              class="content"
                              v-if="
                                dataInfo.numColumns.$columns.includes(column[0])
                              "
                            >
                              <div
                                v-for="column in describeColumn(column[0])"
                                :key="column.index"
                              >
                                <div class="columns">
                                  <div class="column py-2">
                                    {{ column["key"] }}
                                  </div>
                                  <div class="column py-2">
                                    {{ column["value"] }}
                                  </div>
                                </div>
                              </div>
                            </div>
                            <div class="content" v-else>
                              <div>
                                <div class="columns">
                                  <div class="column py-2">count</div>
                                  <div class="column py-2">
                                    {{ df.column(column[0]).count() }}
                                  </div>
                                </div>
                              </div>
                              <div>
                                <div class="columns">
                                  <div class="column py-2">unique values</div>
                                  <div class="column py-2">
                                    {{
                                      df.column(column[0]).unique().$data.length
                                    }}
                                  </div>
                                </div>
                              </div>
                              <div>
                                <div class="columns">
                                  <div class="column py-2">
                                    most frequent value
                                  </div>
                                  <div class="column py-2">
                                    {{
                                      df
                                        .column(column[0])
                                        .valueCounts()
                                        .sortValues({ ascending: false })
                                        .$index[0]
                                    }}
                                  </div>
                                </div>
                              </div>
                              <div>
                                <div class="columns">
                                  <div class="column py-2">frequency</div>
                                  <div class="column py-2">
                                    {{
                                      df
                                        .column(column[0])
                                        .valueCounts()
                                        .sortValues({ ascending: false })
                                        .$data[0]
                                    }}
                                  </div>
                                </div>
                              </div>
                            </div>
                          </div>
                        </div>
                      </div>
                      <div class="column is-half" v-if="column.length > 1">
                        <div class="card">
                          <div class="card-image">
                            <div
                              :id="'plot_' + column[1].toLowerCase()"
                              v-if="!highCardinalityColumns.includes(column[1])"
                            >
                              <!-- Rendered plot goes here -->
                            </div>
                            <div v-else>
                              <figure class="image">
                                <img
                                  style="
                                    position: relative;
                                    height: 450px;
                                    width: 100%;
                                  "
                                  src="https://res.cloudinary.com/dayqxxsip/image/upload/v1690498677/too-many-unique-values-warning_ggrjws.png"
                                  alt="Placeholder image"
                                />
                              </figure>
                            </div>
                          </div>
                          <div class="card-content dx-card-content">
                            <div class="media">
                              <div class="media-left">
                                <figure class="image is-48x48">
                                  <img
                                    v-if="
                                      dataInfo.numColumns.$columns.includes(
                                        column[1]
                                      )
                                    "
                                    src="https://res.cloudinary.com/dayqxxsip/image/upload/v1601923068/Shared/1200px-Greek_uc_sigma.svg_hsngam.png"
                                  />
                                  <img
                                    v-else
                                    src="https://res.cloudinary.com/dayqxxsip/image/upload/v1601923065/Shared/category-icon-png-2_jhizjk.png"
                                  />
                                </figure>
                              </div>
                              <div class="media-content">
                                <p class="title is-4">{{ column[1] }}</p>
                                <p
                                  v-if="
                                    dataInfo.numColumns.$columns.includes(
                                      column[1]
                                    )
                                  "
                                  class="subtitle is-6"
                                >
                                  Numeric variable
                                </p>
                                <p v-else class="subtitle is-6">
                                  Categoric variable
                                </p>
                              </div>
                            </div>

                            <div
                              class="content"
                              v-if="
                                dataInfo.numColumns.$columns.includes(column[1])
                              "
                            >
                              <div
                                v-for="column in describeColumn(column[1])"
                                :key="column.index"
                              >
                                <div class="columns">
                                  <div class="column py-2">
                                    {{ column["key"] }}
                                  </div>
                                  <div class="column py-2">
                                    {{ column["value"] }}
                                  </div>
                                </div>
                              </div>
                            </div>
                            <div class="content" v-else>
                              <div>
                                <div class="columns">
                                  <div class="column py-2">count</div>
                                  <div class="column py-2">
                                    {{ df.column(column[1]).count() }}
                                  </div>
                                </div>
                              </div>
                              <div>
                                <div class="columns">
                                  <div class="column py-2">unique values</div>
                                  <div class="column py-2">
                                    {{
                                      df.column(column[1]).unique().$data.length
                                    }}
                                  </div>
                                </div>
                              </div>
                              <div>
                                <div class="columns">
                                  <div class="column py-2">
                                    most frequent value
                                  </div>
                                  <div class="column py-2">
                                    {{
                                      df
                                        .column(column[1])
                                        .valueCounts()
                                        .sortValues({ ascending: false })
                                        .$index[0]
                                    }}
                                  </div>
                                </div>
                              </div>
                              <div>
                                <div class="columns">
                                  <div class="column py-2">frequency</div>
                                  <div class="column py-2">
                                    {{
                                      df
                                        .column(column[1])
                                        .valueCounts()
                                        .sortValues({ ascending: false })
                                        .$data[0]
                                    }}
                                  </div>
                                </div>
                              </div>
                            </div>
                          </div>
                        </div>
                      </div>
                    </div>
                  </div>
                </b-tab-item>
                <b-tab-item label="Correlation">
                  <div class="columns">
                    <div id="correlation-heatmap-column" class="column">
                      <div id="correlation-heatmap">
                        <!-- Plotly Heatmap -->
                      </div>
                      <div class="columns">
                        <div class="column">
                          <table class="table">
                            <thead>
                              <tr>
                                <th>
                                  Pearson correlation coefficient (<em>r</em>)
                                  value
                                </th>
                                <th>Strength</th>
                                <th>Direction</th>
                              </tr>
                            </thead>
                            <tbody>
                              <tr>
                                <td>Greater than .5</td>
                                <td>Strong</td>
                                <td>Positive</td>
                              </tr>
                              <tr>
                                <td>Between .3 and .5</td>
                                <td>Moderate</td>
                                <td>Positive</td>
                              </tr>
                              <tr>
                                <td>Between 0 and .3</td>
                                <td>Weak</td>
                                <td>Positive</td>
                              </tr>
                              <tr>
                                <td>0</td>
                                <td>None</td>
                                <td>None</td>
                              </tr>
                              <tr>
                                <td>Between 0 and –.3</td>
                                <td>Weak</td>
                                <td>Negative</td>
                              </tr>
                              <tr>
                                <td>Between –.3 and –.5</td>
                                <td>Moderate</td>
                                <td>Negative</td>
                              </tr>
                              <tr>
                                <td>Less than –.5</td>
                                <td>Strong</td>
                                <td>Negative</td>
                              </tr>
                            </tbody>
                          </table>
                        </div>
                        <div class="column">
                          <h3>Highly correlated columns</h3>
                          <small>
                            Remember correlation is not causation. Just because
                            two variables are showing high correlation does not
                            mean there is a causal link and one causes the other
                            one!
                          </small>
                          <br />
                          <br />
                          <ul>
                            <li
                              v-for="message in highCorrelatedMessages"
                              :key="message.id"
                            >
                              {{ message }}
                            </li>
                          </ul>
                        </div>
                      </div>
                    </div>
                  </div>
                </b-tab-item>
                <b-tab-item label="Relationships">
                  <b-tabs
                    :animated="false"
                    size="is-small"
                    class="block"
                    v-model="relationships.variableA"
                    v-if="this.relationshipsInitialised"
                  >
                    <b-tab-item
                      v-for="column in relationships.columns"
                      :key="column.id"
                      :label="column"
                    >
                    </b-tab-item>
                  </b-tabs>
                  <b-tabs
                    :animated="false"
                    size="is-small"
                    class="block"
                    v-model="relationships.variableB"
                    v-if="this.relationshipsInitialised"
                  >
                    <b-tab-item
                      v-for="column in relationships.columnsReversed"
                      :key="column.id"
                      :label="column"
                    >
                    </b-tab-item>
                  </b-tabs>
                  <!-- <b-field grouped>
                        <b-switch v-model="isSwitched" 
                                  v-show="showBreakdownSwitch"
                                  type='is-info'>
                            Show breakdown <table></table>
                        </b-switch>
                        <b-switch v-model="isSwitched">
                            Show as counts
                        </b-switch>
                    </b-field> -->
                  <div class="content has-text-centered">
                    <p v-if="isLoading">
                      <svg
                        xmlns="http://www.w3.org/2000/svg"
                        width="75"
                        height="75"
                        stroke="#000"
                        viewBox="0 0 24 24"
                      >
                        <g class="spinner_V8m1">
                          <circle
                            cx="12"
                            cy="12"
                            r="9.5"
                            fill="none"
                            stroke-width="3"
                          />
                        </g>
                      </svg>
                    </p>
                  </div>
                  <div id="relationship-plot">
                    <!-- Relationship plot -->
                  </div>
                  <div id="relationship-table" v-show="showRelationshipTable">
                    <!-- Relationship table -->
                  </div>
                </b-tab-item>
                <!-- <b-tab-item label="Outliers">
                    outliers
                  </b-tab-item>
                  <b-tab-item label="Missing">
                    missing values
                  </b-tab-item> -->
                <b-tab-item label="Influencers">
                  <div class="has-text-centered">
                    <b-dropdown
                      v-model="selectedInfluencerVariable"
                      aria-role="list"
                      @change="getUniqueInfluencerValues()"
                      scrollable
                      max-height="200"
                    >
                      <template #trigger="{ active }">
                        <b-button
                          :label="
                            'Select target variable: ' +
                            selectedInfluencerVariable
                          "
                          type="is-info"
                          :icon-right="active ? 'menu-up' : 'menu-down'"
                        />
                      </template>
                      <b-dropdown-item
                        v-for="column in influencerColumns"
                        :key="column.id"
                        :value="column"
                        aria-role="listitem"
                      >
                        {{ column }}
                      </b-dropdown-item>
                    </b-dropdown>
                    <b-dropdown
                      v-model="selectedInfluencerValue"
                      aria-role="list"
                      @change="generateInfluencerResults()"
                      scrollable
                      max-height="200"
                    >
                      <template #trigger="{ active }">
                        <b-button
                          :label="
                            'Select target value: ' + selectedInfluencerValue
                          "
                          type="is-info"
                          :icon-right="active ? 'menu-up' : 'menu-down'"
                        />
                      </template>

                      <b-dropdown-item
                        v-for="(value, i) in possibleInfluencerValues"
                        :key="i"
                        :value="value"
                        aria-role="listitem"
                      >
                        {{ value }}
                      </b-dropdown-item>
                    </b-dropdown>
                  </div>
                  <div class="mt-4" style="width: 100%">
                    <div
                      class="box has-text-centered"
                      v-for="result in influencerResults"
                      :key="result.id"
                      v-html="result"
                    ></div>
                  </div>
                </b-tab-item>
              </b-tabs>
            </div>
          </div>
        </div>
      </article>
    </div>
  </div>
</template>

<script>
// How to use Chart.js in Vue.js and Nuxt
// https://stackoverflow.com/questions/67197212/vue-chartjs-you-may-need-an-appropriate-loader-to-handle-this-file-type-error
// https://stackoverflow.com/questions/66940954/why-does-nuxt-give-me-this-error-with-vue-chartjs

// Reading CSV data
// https://stackoverflow.com/questions/67342511/how-to-get-the-data-inside-of-csv-file

import * as dfd from "danfojs";
import Plotly from "plotly.js-dist-min";
import axios from "axios";

export default {
  data() {
    return {
      dropFiles: [],
      isFullPage: true,
      isLoading: false,
      df: null,
      dummies: null,
      dataInfo: {
        columns: [],
        warnings: [],
      },
      allHighCardinalityColumns: [], // This one includes numeric columns
      highCardinalityColumns: [], // This one is only categoric columns
      highCorrelatedMessages: [],
      variablePlotsInitialised: false,
      heatmapInitialised: false,
      relationshipsInitialised: false,
      influencerColumns: [],
      selectedInfluencerVariable: "",
      selectedInfluencerValue: "",
      possibleInfluencerValues: [],
      influencerResults: [],
      isSwitched: false,
      showBreakdownSwitch: false,
      showRelationshipTable: false,
      activeTab: 0,
      relationships: {
        variableA: 0,
        variableB: 1,
        columns: [],
        columnsReversed: [],
      },
    };
  },
  methods: {
    fetchTitanicData() {
      const url = 'https://raw.githubusercontent.com/MainakRepositor/Datasets/master/Titanic.csv';

      this.isLoading = true;
      axios.get(url)
        .then(response => {
          console.log("response", response);
          const blob = new Blob([response.data], { type: 'text/csv' });
          const file = new File([blob], "Titanic.csv", { type: "text/csv" });
          console.log("file", file);

          this.dropFiles = file;
        })
        .catch(error => {
          console.error('Error fetching Titanic dataset:', error);
        })
        .finally(() => {
          this.isLoading = false;
        });
    },
    deleteDropFile() {
      this.dropFiles = [];
      this.variablePlotsInitialised = false;
      this.heatmapInitialised = false;
      this.relationshipsInitialised = false;
      this.relationships = {
        variableA: 0,
        variableB: 1,
        columns: [],
        columnsReversed: [],
      };
    },
    tabClicked(index) {
      if (index === 2) {
        this.generateVariablesPlots();
      }

      if (index === 3) {
        this.generateHeatmap();
      }

      if (index === 4) {
        this.initialiseRelationships();
      }
    },
    generateVariablesPlots() {
      if (this.variablePlotsInitialised == true) {
        return;
      }

      let layout = this.getChartLayout("Count");

      // Plot numeric columns univariate charts
      for (let column of this.dataInfo["numColumns"].$columns) {
        this.df[column].plot("plot_" + column.toLowerCase()).hist({ layout });
      }

      // Plot categoric columns univariate charts
      for (let column of this.dataInfo["catColumns"].$columns) {
        if (this.highCardinalityColumns.includes(column)) {
          continue;
        }

        let counts = this.df.column(column).valueCounts();
        let df = new dfd.DataFrame([counts.$data], { columns: counts.$index });
        df.plot("plot_" + column.toLowerCase()).bar({ layout });
      }

      this.variablePlotsInitialised = true;
    },
    /*
     * Creates a correlation heatmap
     * https://plotly.com/javascript/heatmaps/
     */
    generateHeatmap() {
      /**
       * This needs to be in the format of
       *  zValues = [
       *     [0.00, 0.00, 0.75, 0.75, 1.00],
       *     [0.00, 0.00, 0.75, 1.00, 0.00],
       *     [0.75, 0.75, 1.00, 0.75, 0.75],
       *     [0.00, 1.00, 0.00, 0.75, 0.00],
       *     [1.00, 0.00, 0.00, 0.75, 0.00]
       *  ];
       */
      if (this.heatmapInitialised) {
        return;
      }

      let zValues = [];
      let df = this.df.copy();
      let columnsLength = this.dataInfo.columns.length;
      let columnsToDrop = [];

      // Drop columns with high cardinality (many unique values)
      for (let i = 0; i < columnsLength; i++) {
        let column = this.dataInfo.columns[i];

        // Skip if a numeric column as it will have lots of unique values
        // but this doesn't matter :)
        if (this.dataInfo["numColumns"].$columns.includes(column)) {
          continue;
        }

        let uniqueValuesCount = df.column(column).unique().$data.length;

        if (uniqueValuesCount > 5) {
          columnsToDrop.push(column);
        }
      }

      df.drop({ columns: columnsToDrop, inplace: true });

      // Create dummy columns for categoric variables
      let dummies = dfd.getDummies(df);
      // Uncomment to debug: console.log("DUMMIES", dummies);
      columnsLength = dummies.$columns.length;

      for (let i = 0; i < columnsLength; i++) {
        let column = dummies.$columns[i];
        // Uncomment to debug: console.log("COMPARING", column);
        let correlations = [];

        for (let j = 0; j < columnsLength; j++) {
          let comparisonColumn = dummies.$columns[j];
          // Uncomment to debug: console.log("TO", comparisonColumn);

          let pearsonCorrelation = this.corr(
            dummies[column].$data,
            dummies[comparisonColumn].$data
          ).toFixed(2);

          correlations.push(pearsonCorrelation);

          this.logCorrelationMessage(
            pearsonCorrelation,
            column,
            comparisonColumn
          );
        }

        zValues.push(correlations);
      }

      var xValues = dummies.$columns;
      var yValues = dummies.$columns;

      var colorscaleValue = [
        [0, "#3D9970"],
        [1, "#001f3f"],
      ];

      var data = [
        {
          x: xValues,
          y: yValues,
          z: zValues,
          type: "heatmap",
          colorscale: colorscaleValue,
          showscale: false,
        },
      ];

      var columnWidth = document.getElementById("main-column").offsetWidth - 35;

      if (!columnWidth > 0) {
        columnWidth = window.innerWidth - 750;
      }

      var layout = {
        autosize: false,
        width: columnWidth,
        height: 700,
        annotations: [],
        xaxis: {
          ticks: "",
          side: "top",
        },
        yaxis: {
          ticks: "",
          ticksuffix: " ",
          autosize: false,
        },
      };

      for (var i = 0; i < yValues.length; i++) {
        for (var j = 0; j < xValues.length; j++) {
          var currentValue = zValues[i][j];
          if (currentValue != 0.0) {
            var textColor = "white";
          } else {
            var textColor = "black";
          }
          var result = {
            xref: "x1",
            yref: "y1",
            x: xValues[j],
            y: yValues[i],
            text: zValues[i][j],
            font: {
              family: "Arial",
              size: 12,
              color: "rgb(50, 171, 96)",
            },
            showarrow: false,
            font: {
              color: textColor,
            },
          };
          layout.annotations.push(result);
        }
      }

      Plotly.newPlot("correlation-heatmap", data, layout);

      this.heatmapInitialised = true;
    },
    initialiseRelationships() {
      if (this.relationshipsInitialised) {
        return;
      }

      this.isLoading = true;

      setTimeout(() => {
        // Present those columns as the column names to relationships selector tabs
        // Remove columns with lots of unique categoric values
        for (let column of this.df.$columns) {
          let isNumericColumn =
            this.dataInfo.numColumns.$columns.includes(column);
          let hasLowUniqueValues =
            this.df.column(column).unique().$data.length < 8;

          if (isNumericColumn || hasLowUniqueValues) {
            this.relationships.columns.push(column);
            this.relationships.columnsReversed.push(column);
          }
        }

        // Present initial scatter chart
        this.generateRelationshipPlot();
        this.isLoading = false;

        // Watcher can then update on further changes
        this.relationshipsInitialised = true;
      }, 500);
    },
    generateRelationshipPlot() {
      let variableA = this.relationships.columns[this.relationships.variableA];
      let variableB = this.relationships.columns[this.relationships.variableB];

      let numColumns = this.dataInfo["numColumns"].$columns;
      let catColumns = this.dataInfo["catColumns"].$columns;

      let bothVariablesNumeric =
        numColumns.includes(variableA) && numColumns.includes(variableB);
      let bothVariablesCategoric =
        catColumns.includes(variableA) && catColumns.includes(variableB);

      if (bothVariablesNumeric) {
        this.df.plot("relationship-plot").scatter({
          config: { x: variableA, y: variableB },
        });

        if (variableA != variableB) {
          this.showRelationshipTable = true;

          setTimeout(() => {
            let group = this.df
              .loc({ columns: [variableA, variableB] })
              .groupby([variableA]);

            let count = group.count();
            let sum = group.sum();
            let mean = group.mean();

            let merge_df = dfd.merge({
              left: count,
              right: sum,
              on: [variableA],
              how: "inner",
            });

            merge_df = dfd.merge({
              left: merge_df,
              right: mean,
              on: [variableA],
              how: "inner",
            });

            merge_df.plot("relationship-table").table();
          }, 300);
        }
      } else if (bothVariablesCategoric) {
        if (variableA == variableB) {
          return;
        }

        let group = this.df
          .loc({ columns: [variableA, variableB] })
          .groupby([variableA, variableB])
          .size();

        group.$columns[group.$columns.length - 1] = "Count";

        group.plot("relationship-plot").table();
        this.showRelationshipTable = false;
      } else {
        if (variableA == variableB) {
          return;
        }

        this.showRelationshipTable = true;

        setTimeout(() => {
          let categoricColumn;
          let numericColumn;

          if (catColumns.includes(variableA)) {
            categoricColumn = variableA;
            numericColumn = variableB;
          } else {
            categoricColumn = variableB;
            numericColumn = variableA;
          }

          let numericGrouped = this.df
            .loc({ columns: [categoricColumn, numericColumn] })
            .groupby([categoricColumn, numericColumn])
            .size();

          numericGrouped.$columns[numericGrouped.$columns.length - 1] = "Count";

          let group = this.df
            .loc({ columns: [categoricColumn, numericColumn] })
            .groupby([categoricColumn]);

          let count = group.count();
          let sum = group.sum();
          let mean = group.mean();

          let merge_df = dfd.merge({
            left: count,
            right: sum,
            on: [categoricColumn],
            how: "inner",
          });

          merge_df = dfd.merge({
            left: merge_df,
            right: mean,
            on: [categoricColumn],
            how: "inner",
          });

          merge_df.plot("relationship-plot").table();

          numericGrouped.plot("relationship-table").table();
        }, 300);
      }
    },
    getUniqueInfluencerValues() {
      setTimeout(() => {
        this.selectedInfluencerValue = "";
        this.possibleInfluencerValues = [];

        let isNumericVariable = this.dataInfo["numColumns"].$columns.includes(
          this.selectedInfluencerVariable
        );

        let isHighCardinalityVariable = this.allHighCardinalityColumns.includes(
          this.selectedInfluencerVariable
        );

        if (isNumericVariable) {
          this.possibleInfluencerValues = ["Higher", "Lower"];
        }

        if (!isNumericVariable || !isHighCardinalityVariable) {
          let uniqueValues = this.df
            .column(this.selectedInfluencerVariable)
            .unique().$data;
          this.possibleInfluencerValues = [...uniqueValues].sort();
        }
      }, 300);
    },
    generateInfluencerResults() {
      setTimeout(() => {
        if (
          this.selectedInfluencerVariable.toString() == "" ||
          this.selectedInfluencerValue.toString() == ""
        ) {
          return;
        }

        this.influencerResults = [];

        console.log(
          "Generating influencer results for " +
            this.selectedInfluencerVariable +
            " with the value " +
            this.selectedInfluencerValue
        );

        if (
          this.selectedInfluencerValue == "Higher" ||
          this.selectedInfluencerValue == "Lower"
        ) {
          return this.generateContinuousInfluencerResults();
        }

        let query_df = this.df.query(
          this.df[this.selectedInfluencerVariable].eq(
            this.selectedInfluencerValue
          )
        );

        for (let column of query_df.$columns) {
          if (column == this.selectedInfluencerVariable) {
            continue;
          }

          if (this.highCardinalityColumns.includes(column)) {
            continue;
          }

          if (this.dataInfo.numColumns.$columns.includes(column)) {
            this.influencerResults.push(
              "<b>" +
                column +
                "</b> has an average of <b>" +
                query_df.column(column).mean().toFixed(2) +
                "</b>"
            );
          }

          if (!this.allHighCardinalityColumns.includes(column)) {
            let valueCounts = query_df
              .column(column)
              .valueCounts()
              .sortValues({ ascending: false });

            let topValueCount = valueCounts.$data[0];
            let topValueLabel = valueCounts.$index[0];
            let percentage = (
              (topValueCount / this.dataInfo.numberOfRows) *
              100
            ).toFixed(2);

            this.influencerResults.push(
              "<b>" +
                column +
                "</b> most frequent value is <b>" +
                topValueLabel +
                "</b> with " +
                topValueCount +
                " records (" +
                percentage +
                "% of the data)"
            );
          }
        }
      }, 300);
    },
    generateContinuousInfluencerResults() {
      let query_df;
      let mean = this.df.column(this.selectedInfluencerVariable).mean();

      if (this.selectedInfluencerValue == "Higher") {
        query_df = this.df.query(
          this.df[this.selectedInfluencerVariable].gt(mean * 1.2) // 20% higher than mean
        );
      } else {
        query_df = this.df.query(
          this.df[this.selectedInfluencerVariable].lt(mean * 1.2) // 20% lower than mean
        );
      }

      for (let column of query_df.$columns) {
        if (column == this.selectedInfluencerVariable) {
          continue;
        }

        if (this.highCardinalityColumns.includes(column)) {
          continue;
        }

        if (this.dataInfo.numColumns.$columns.includes(column)) {
          this.influencerResults.push(
            "<b>" +
              column +
              "</b> has an average of <b>" +
              query_df.column(column).mean().toFixed(2) +
              "</b>"
          );
        }

        if (!this.allHighCardinalityColumns.includes(column)) {
          let valueCounts = query_df
            .column(column)
            .valueCounts()
            .sortValues({ ascending: false });

          let topValueCount = valueCounts.$data[0];
          let topValueLabel = valueCounts.$index[0];
          let percentage = (
            (topValueCount / this.dataInfo.numberOfRows) *
            100
          ).toFixed(2);

          this.influencerResults.push(
            "<b>" +
              column +
              "</b> most frequent value is <b>" +
              topValueLabel +
              "</b> with " +
              topValueCount +
              " records (" +
              percentage +
              "% of the data)"
          );
        }
      }
    },
    logCorrelationMessage(pearsonCorrelation, columnName, comparisonColumn) {
      if (columnName == comparisonColumn) {
        return;
      }

      let messageAlreadyExists =
        this.highCorrelatedMessages.filter((str) =>
          str.includes(`${comparisonColumn} and ${columnName}`)
        ).length > 0;

      if (messageAlreadyExists) {
        return;
      }

      if (pearsonCorrelation > 0.5) {
        // Strong Positive
        this.highCorrelatedMessages.push(
          `${columnName} and ${comparisonColumn} have a strong positive correlation 
            of ${pearsonCorrelation}`
        );
      }

      if (pearsonCorrelation > 0.3 && pearsonCorrelation < 0.5) {
        // Moderate Positive
        this.highCorrelatedMessages.push(
          `${columnName} and ${comparisonColumn} have a moderate positive correlation 
            of ${pearsonCorrelation}`
        );
      }

      if (pearsonCorrelation < -0.5) {
        // Strong Negative
        this.highCorrelatedMessages.push(
          `${columnName} and ${comparisonColumn} have a strong negative correlation 
            of ${pearsonCorrelation}`
        );
      }

      if (pearsonCorrelation < -0.3 && pearsonCorrelation > -0.5) {
        // Moderate Negative
        this.highCorrelatedMessages.push(
          `${columnName} and ${comparisonColumn} have a moderate negative correlation 
            of ${pearsonCorrelation}`
        );
      }
    },
    getChartLayout(title) {
      let section = document.getElementsByClassName("tab-content")
      let sectionWidth = section[0].offsetWidth;
      let plotWidth = (sectionWidth / 2) - 30;

      return {
        width: plotWidth,
        yaxis: {
          title: title,
        },
        xaxis: {
          title: "",
        },
      };
    },
    describeColumn(column) {
      let description = this.df.column(column).describe();
      let result = [];

      for (let i = 0; i < description.$index.length; i++) {
        result.push({
          key: description.$index[i],
          value: description.$data[i],
        });
      }

      return result;
    },
    /**
     * Chunks an array into sub-arrays of a given size.
     * @param {Array} array The array to chunk.
     * @param {number} size The size of each chunk.
     * @returns {Array} An array of chunks.
     */
    chunk(array, size) {
      let result = [];
      let count = 0;

      for (let i = 0; i < Math.ceil(array.length) / size; i++) {
        let start = i * size;
        let end = start + size;
        result.push(array.slice(start, end));
        count += 1;
      }

      return result;
    },
    /*
     * Calculates Pearson correlation between
     * two arrays x and y.
     */
    corr(x, y) {
      let sumX = 0,
        sumY = 0,
        sumXY = 0,
        sumX2 = 0,
        sumY2 = 0;

      const minLength = (x.length = y.length = Math.min(x.length, y.length)),
        reduce = (xi, idx) => {
          const yi = y[idx];
          sumX += xi;
          sumY += yi;
          sumXY += xi * yi;
          sumX2 += xi * xi;
          sumY2 += yi * yi;
        };

      x.forEach(reduce);

      return (
        (minLength * sumXY - sumX * sumY) /
        Math.sqrt(
          (minLength * sumX2 - sumX * sumX) * (minLength * sumY2 - sumY * sumY)
        )
      );
    },
  },
  watch: {
    /*
     Reads the given CSV file when the upload input
     changes. Parses the CSV and calculates dataset 
     statistics.
     */
    dropFiles() {
      if (this.dropFiles == null) {
        return;
      }

      if (this.dropFiles.length < 1) {
        return;
      }

      console.log("this.dropFiles", this.dropFiles);
      console.log("this.dropFiles[0]", this.dropFiles[0])

      const reader = new FileReader();

      reader.readAsText(this.dropFiles);

      reader.onload = () => {
        this.rawCSVData = reader.result; // contains the file content as a string
      };

      console.log("this.rawCSVData", this.rawCSVData);

      reader.onerror = () => {
        console.log(reader.error);
      };

      dfd.readCSV(this.dropFiles).then((df) => {
        let dfShape = df.shape;

        let numericColumns = df.selectDtypes(["int32", "float32"]);

        let categoricalColumns = df.selectDtypes(["string"]);
        let totalMissingValues = df
          .isNa()
          .sum()
          .values.reduce((a, b) => a + b, 0);

        let rows = [];
        let duplicateRows = [];

        for (let i = 0; i < dfShape[0]; i++) {
          let row = df.iloc({ rows: [i] }).$data[0];

          if (rows.includes(JSON.stringify(row))) {
            duplicateRows.push(row);
          }

          rows.push(JSON.stringify(row));
        }

        let dataInfo = {
          numberOfRows: dfShape[0],
          numberOfColumns: dfShape[1],
          columns: df.columns,
          dtypes: df.ctypes,
          numColumns: numericColumns,
          catColumns: categoricalColumns,
          missingValuesByColumn: df.isNa(),
          totalMissingValues: totalMissingValues,
          "missingValues%": (
            (totalMissingValues / (dfShape[0] * dfShape[1])) *
            100
          ).toFixed(2),
          duplicateRows: duplicateRows.length,
          "duplicateRows%": ((duplicateRows.length / dfShape[0]) * 100).toFixed(
            2
          ),
        };

        console.log("dataInfo", dataInfo);

        // Identify high cardinality categoric columns
        this.influencerColumns = [...dataInfo["numColumns"].columns];
        for (let column of dataInfo["catColumns"].$columns) {
          let uniqueValuesCount = df.column(column).unique().$data.length;

          if (uniqueValuesCount > 10) {
            this.highCardinalityColumns.push(column);
          } else {
            this.influencerColumns.push(column);
          }
        }

        let warnings = [];

        for (let column of dataInfo["columns"]) {
          let missingValues =
            dataInfo["missingValuesByColumn"][column].valueCounts();
          let uniqueValuesCount = df.column(column).unique().$data.length;

          if (uniqueValuesCount == 2) {
            warnings.push(
              `<b>${column}</b> appears to be a binary variable: 2 values`
            );
          }

          if (uniqueValuesCount > 10) {
            warnings.push(
              `<b>${column}</b> has high cardinality: ${uniqueValuesCount} unique values`
            );
            this.allHighCardinalityColumns.push(column);
          }

          if (missingValues.$dataIncolumnFormat.length > 1) {
            let missingValuesCount = missingValues.$dataIncolumnFormat[1];
            let missingValuesPc = (
              (missingValuesCount / dataInfo["numberOfRows"]) *
              100
            ).toFixed(2);

            warnings.push(
              `<b>${column}</b> has ${missingValuesCount} (${missingValuesPc}%) missing values`
            );
          }
        }

        this.dataInfo = dataInfo;
        this.dataInfo.warnings = warnings;
        this.df = df;
        this.activeTab = 0;

        this.isLoading = false;
      });
    },
    relationships: {
      handler: function (newValue) {
        this.generateRelationshipPlot();
      },
      deep: true,
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss">
body,
button,
input,
optgroup,
select,
textarea {
  font-family: BlinkMacSystemFont, -apple-system, "Segoe UI", Roboto, Oxygen,
    Ubuntu, Cantarell, "Fira Sans", "Droid Sans", "Helvetica Neue", Helvetica,
    Arial, sans-serif;
}

article {
  margin-top: 20px;

  p {
    font-size: 20px;
  }
}

article figcaption > a,
article p > a {
  color: #4a4a4a;
  text-decoration: underline;
  font-weight: 600;
}

article h1,
article h2,
article h3,
article h4,
article h5,
article h6 {
  margin: 30px 0 10px;
  color: rgb(74, 74, 74);
  padding: 20px 0 0;
  font-weight: 700;
  -webkit-font-smoothing: antialiased;
  cursor: text;
  position: relative;
}

article figcaption > a:hover,
article p > a:hover {
  text-decoration: none;
}

article .dx a:hover {
  color: #363636;
}

article figcaption > a:active,
article figcaption > a:focus,
article p > a:active,
article p > a:focus {
  border-bottom: 2px solid #4a4a4a;
  text-decoration: none;
}

.spinner_V8m1 {
  transform-origin: center;
  animation: spinner_zKoa 2s linear infinite;
}
.spinner_V8m1 circle {
  stroke-linecap: round;
  animation: spinner_YpZS 1.5s ease-in-out infinite;
}
@keyframes spinner_zKoa {
  100% {
    transform: rotate(360deg);
  }
}
@keyframes spinner_YpZS {
  0% {
    stroke-dasharray: 0 150;
    stroke-dashoffset: 0;
  }
  47.5% {
    stroke-dasharray: 42 150;
    stroke-dashoffset: -16;
  }
  95%,
  100% {
    stroke-dasharray: 42 150;
    stroke-dashoffset: -59;
  }
}
article .dx {
  margin-bottom: 250px;
}

article .dx p,
article .dx blockquote,
article .dx ul,
article .dx ol,
article .dx dl,
article .dx li,
article .dx table,
article .dx pre {
  margin: 0px;
}

article .dx input {
  display: none;
}

article .dx li a {
  text-decoration: none;
  font-weight: 600;
}

.tabs li.is-active a {
    border-bottom-color: #373737 !important;
    color: #373737 !important;
    border-bottom: 5px solid #373737 !important;
}

.tabs li a:hover {
  border-bottom-color: #373737;
  color: #373737;
  border-bottom: 5px solid #373737;
}

article .dx ul,
article .dx ol {
  padding-left: 0px;
  font-size: 16px;
}

article .dx table tr {
  font-size: 16px !important;
}

article .dx-card-content {
  min-height: 315px;
}

.dx-thead {
  font-size: 20px !important;
}

.upload .upload-draggable {
  border-color: rgb(69, 75, 78);
  min-width: 700px;
  margin-top: 50px;
}

.upload .upload-draggable {
  cursor: pointer;
  padding: 0.25em;
  border: 1px dashed #b5b5b5;
  border-radius: 6px;
}

.upload .upload-draggable.is-hovered.is-primary,
.upload .upload-draggable:hover.is-primary {
  border-color: rgb(60, 33, 132);
  background-image: initial;
  background-color: rgba(64, 35, 142, 0.05);
}

#privacy-message-box {
  padding: 0px;
}

.button.is-info {
    background-color: #3e8ed0 !important;
    border-color: transparent;
    color: #fff;
}

.dropdown .dropdown-menu .has-link a.is-active, a.dropdown-item.is-active, button.dropdown-item.is-active {
  background-color: #485fc7 !important;
  color: #fff;
}
</style>