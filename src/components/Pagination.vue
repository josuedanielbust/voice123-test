<template>
 <ul class="pagination">
    <li class="pagination__item" >
      <button type="button" @click="onClickPreviousPage" :disabled="isInFirstPage"
              aria-label="Go to previous page">Previous</button>
    </li>

    <li v-for="page in pages" :key="page.name" class="pagination-item">
      <button type="button" @click="onClickPage(page.name)" :disabled="page.isDisabled"
              :class="{ active: isPageActive(page.name) }"
              :aria-label="`Go to page number ${page.name}`">{{ page.name }}</button>
    </li>

    <li class="pagination__item">
      <button type="button" @click="onClickNextPage" :disabled="isInLastPage"
              aria-label="Go to next page">Next</button>
    </li>
  </ul>
</template>

<script>
export default {
  name: 'Pagination',
  props: {
    maxVisibleButtons: {
      type: Number,
      required: false,
      default: 10,
    },
    totalPages: {
      type: Number,
      required: true,
    },
    total: {
      type: Number,
      required: true,
    },
    perPage: {
      type: Number,
      required: true,
    },
    currentPage: {
      type: Number,
      required: true,
    },
    url: {
      type: String,
      required: true,
    },
    keywords: {
      type: String,
      required: true,
    },
  },
  computed: {
    startPage() {
      if (this.currentPage === 1) { return 1; }
      if (this.currentPage === this.totalPages) {
        return this.totalPages - this.maxVisibleButtons + 1;
      }
      return this.currentPage - 1;
    },
    endPage() {
      return Math.min(this.startPage + this.maxVisibleButtons - 1, this.totalPages);
    },
    pages() {
      const range = [];
      for (let i = this.startPage; i <= this.endPage; i += 1) {
        range.push({
          name: i,
          isDisabled: i === this.currentPage,
        });
      }
      return range;
    },
    isInFirstPage() { return this.currentPage === 1; },
    isInLastPage() { return this.currentPage === this.totalPages; },
  },
  methods: {
    onClickFirstPage() { this.$emit('pagechanged', 1); },
    onClickPreviousPage() { this.$emit('pagechanged', this.currentPage - 1); },
    onClickPage(page) {
      this.$parent.runQuery(page, page);
      this.$emit('pagechanged', page);
    },
    onClickNextPage() { this.$emit('pagechanged', this.currentPage + 1); },
    onClickLastPage() { this.$emit('pagechanged', this.totalPages); },
    isPageActive(page) { return this.currentPage === page; },
  },
};
</script>

<style lang="scss">
ul.pagination {
  $black: #000000;
  $white: #ffffff;
  $color-1: #285bd4;
  $color-2: #85edee;
  $color-3: #cffcf6;

  list-style: none;
  margin: 2rem 0 1rem;
  li {
    display: inline-block;
    padding: 0;
    button {
      cursor: pointer;
      min-height: 2rem;
      max-height: 2rem;
      min-width: 2.5rem;
      max-width: 2.5rem;
      background-color: $white;
      border: 0;
      color: $color-1;
      font-size: 0.8rem;
      &:hover {
        text-decoration: underline;
      }
      &[disabled="disabled"] {
        color: $color-2;
        cursor: auto;
        &:hover {
          text-decoration: none;
        }
      }
    }
    &:first-of-type, &:last-of-type {
      button {
        min-width: 5rem;
        max-width: 5rem;
      }
    }
  }
}
</style>
