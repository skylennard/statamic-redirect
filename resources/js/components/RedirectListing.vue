<template>
  <div>
    <div v-if="initializing" class="card loading">
      <loading-graphic />
    </div>

    <data-list
      v-if="!initializing"
      class="mb-4"
      :visible-columns="columns"
      :columns="columns"
      :rows="items"
      :sort="false"
      :sort-column="sortColumn"
      :sort-direction="sortDirection"
    >
      <div slot-scope="{ hasSelections }">
        <div class="card p-0 relative">
          <data-list-filter-presets
            ref="presets"
            :active-preset="activePreset"
            :preferences-prefix="preferencesPrefix"
            @selected="selectPreset"
            @reset="filtersReset"
          />
          <div class="data-list-header">
            <data-list-filters
              :filters="filters"
              :active-preset="activePreset"
              :active-preset-payload="activePresetPayload"
              :active-filters="activeFilters"
              :active-filter-badges="activeFilterBadges"
              :active-count="activeFilterCount"
              :search-query="searchQuery"
              :saves-presets="true"
              :preferences-prefix="preferencesPrefix"
              @filter-changed="filterChanged"
              @search-changed="searchChanged"
              @saved="$refs.presets.setPreset($event)"
              @deleted="$refs.presets.refreshPresets()"
              @restore-preset="$refs.presets.viewPreset($event)"
              @reset="filtersReset"
            />
          </div>

          <div
            v-show="items.length === 0"
            class="p-3 text-center text-grey-50"
            v-text="__('No results')"
          />

          <data-list-table
            v-show="items.length"
            :allow-bulk-actions="false"
            :loading="loading"
            :reorderable="false"
            :sortable="true"
            :toggle-selection-on-row-click="false"
            :allow-column-picker="true"
            :column-preferences-key="preferencesKey('columns')"
            @sorted="sorted"
          >
            <template slot="cell-enabled" slot-scope="{ row: redirect }">
              <div
                v-if="redirect.enabled"
                class="bg-green block h-3 w-2 mx-auto rounded-full"
              ></div>
              <div
                v-else
                class="bg-red block h-3 w-2 mx-auto rounded-full"
              ></div>
            </template>
            <template slot="cell-source" slot-scope="{ row: redirect }">
              <a :href="cp_url('redirect/redirects/' + redirect.id)">{{
                redirect.source
              }}</a>
            </template>
            <template slot="actions" slot-scope="{ row: redirect, index }">
              <dropdown-list>
                <dropdown-item
                  :text="__('Edit')"
                  :redirect="cp_url('redirect/redirects/' + redirect.id)"
                />
                <dropdown-item
                  :text="__('Delete')"
                  class="warning"
                  @click="confirmDeleteRow(redirect.id, index)"
                />
              </dropdown-list>

              <confirmation-modal
                v-if="deletingRow !== false"
                :title="deletingModalTitle"
                :bodyText="__('Are you sure you want to delete this redirect?')"
                :buttonText="__('Delete')"
                :danger="true"
                @confirm="deleteRow('/redirect/redirects')"
                @cancel="cancelDeleteRow"
              >
              </confirmation-modal>
            </template>
          </data-list-table>
        </div>
      </div>
    </data-list>
    <data-list-pagination
      class="mt-3"
      :resource-meta="meta"
      :per-page="perPage"
      @page-selected="selectPage"
      @per-page-changed="changePerPage"
    />
  </div>
</template>

<script>
import DeletesListingRow from "./DeletesListingRow.js";
import Listing from "../../../vendor/statamic/cms/resources/js/components/Listing";

export default {
  mixins: [Listing, DeletesListingRow],

  data() {
    return {
      listingKey: "redirects",
      preferencesPrefix: `redirect.redirects`,
      requestUrl: cp_url(`redirect/api/redirects`),
      columns: this.columns,
    };
  },
};
</script>
