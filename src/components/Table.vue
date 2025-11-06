<template>
  <div class="table-container">
    <table class="table">
      <thead>
        <tr>
          <th v-for="(col, i) in columns" :key="i">
            {{ col.label }}
          </th>
          <th class="actions-header">Actions</th>
        </tr>
      </thead>
      <tbody v-if="loading" class="skeleton-loader">
        <tr v-for="row in rowsPerPage" :key="row">
          <td v-for="col in columns" :key="col.label">
            <div class="skeleton-box"></div>
          </td>
          <td>
            <div class="action-btns">
              <div class="skeleton-btn"></div>
              <div class="skeleton-btn"></div>
              <div class="skeleton-btn"></div>
            </div>
          </td>
        </tr>
      </tbody>
      <tbody v-else>
        <tr v-for="(row, rowI) in paginatedData" :key="rowI">
          <td
            v-for="(col, colI) in columns"
            :key="colI"
            :data-label="col.label">
            {{ row[col.field] }}
          </td>
          <td data-label="Actions">
            <div class="action-btns">
              <button class="action-btn view" @click="$emit('view', row)">
                View
              </button>
              <button class="action-btn edit" @click="$emit('edit', row)">
                Edit
              </button>
              <button class="action-btn delete" @click="$emit('delete', row)">
                Delete
              </button>
            </div>
          </td>
        </tr>
      </tbody>
    </table>

    <div class="pagination-buttons">
      <button :disabled="currentPage === 1" @click="currentPage--">
        <svg viewBox="0 0 47 42" fill="none" xmlns="http://www.w3.org/2000/svg">
          <path
            d="M46.6699 36.298L41.4845 41.4834L20.7427 20.7416L41.4844 9.80027e-07L46.6698 5.1854L31.1136 20.7416L46.6699 36.298Z"
            fill="currentColor" />
          <path
            d="M20.7423 41.4834L25.9277 36.298L10.3712 20.7416L25.9277 5.1854L20.742 9.80046e-07L0.000364471 20.7416L20.7423 41.4834Z"
            fill="currentColor" />
        </svg>
      </button>
      <span>Page {{ currentPage }} of {{ totalPages }}</span>
      <button :disabled="currentPage === totalPages" @click="currentPage++">
        <svg viewBox="0 0 47 42" fill="none" xmlns="http://www.w3.org/2000/svg">
          <path
            d="M0 5.18544L5.18544 0L25.9272 20.7418L5.18555 41.4834L0.000108401 36.298L15.5563 20.7418L0 5.18544Z"
            fill="currentColor" />
          <path
            d="M25.9276 0L20.7422 5.18544L36.2988 20.7418L20.7422 36.298L25.928 41.4834L46.6696 20.7418L25.9276 0Z"
            fill="currentColor" />
        </svg>
      </button>
    </div>
  </div>
</template>

<script>
export default {
  name: "Table",
  props: {
    columns: {
      type: Array,
      required: true,
    },
    data: {
      type: Array,
      required: true,
    },
    loading: {
      type: Boolean,
      required: true,
    },
  },
  data() {
    return {
      currentPage: 1,
      rowsPerPage: 10,
    };
  },
  computed: {
    totalPages() {
      return Math.ceil(this.data.length / this.rowsPerPage);
    },
    paginatedData() {
      const start = (this.currentPage - 1) * this.rowsPerPage;
      return this.data.slice(start, start + this.rowsPerPage);
    },
  },
  watch: {
    rowsPerPage() {
      this.currentPage = 1;
    },
  },
};
</script>

<style scoped>
.skeleton-loader td {
  padding: 20px 16px;
}

.skeleton-box {
  width: 100%;
  height: 20px;
  background: #e0e0e0;
  position: relative;
  overflow: hidden;
  border-radius: 4px;
}

.skeleton-btn {
  width: 46px;
  height: 28px;
  background: #e0e0e0;
  position: relative;
  overflow: hidden;
  border-radius: 4px;
}

.skeleton-box::before,
.skeleton-btn::before {
  content: "";
  display: block;
  width: 100%;
  height: 100%;
  position: absolute;
  top: 0;
  left: -150%;
  background: linear-gradient(
    90deg,
    transparent,
    rgba(255, 255, 255, 0.5),
    transparent
  );
  animation: shimmer 1.5s infinite;
}

@keyframes shimmer {
  0% {
    left: -150%;
  }
  50% {
    left: 100%;
  }
  100% {
    left: 150%;
  }
}

.table-container {
  width: 100%;
  overflow-x: auto;
  padding: 0 20px;
  margin-bottom: 100px;
}

.table {
  margin: 20px auto;
  border: 1px solid #eee;
  width: 100%;
  border-collapse: collapse;
  background: white;
  border-radius: 8px;
  overflow: hidden;
}

.table th,
.table td {
  padding: 12px 16px;
  text-align: left;
  border-bottom: 1px solid #eee;
}

.table th:first-child,
.table td:first-child,
.table th:nth-child(2),
.table td:nth-child(2) {
  width: 190px;
  min-width: 100px;
  max-width: 190px;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.table tbody tr:hover {
  background: #fbf8fe;
}

.actions-header {
  text-align: center;
}

.action-btns {
  display: flex;
  gap: 8px;
}

.action-btn {
  border: none;
  padding: 6px 10px;
  border-radius: 4px;
  color: white;
  cursor: pointer;
  font-size: 12px;
}

.view {
  background-color: #6007a5;
}

.edit {
  background-color: #00abf4;
}

.delete {
  background-color: #d9534f;
}

.pagination-controls,
.pagination-buttons {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 12px;
}

.pagination-buttons {
  margin-top: 12px;
  justify-content: end;
}

.pagination-buttons button {
  cursor: pointer;
  padding: 6px 12px;
  border-radius: 4px;
  border: none;
  color: #6007a5;
  background-color: transparent;
}

.pagination-buttons button svg {
  width: 18px;
  height: 18px;
}

.pagination-buttons button:disabled {
  color: gray;
  background-color: transparent;
}

/* --- Mobile --- */
@media (max-width: 768px) {
  .table thead {
    display: none;
  }

  .table tr {
    display: block;
    margin-bottom: 12px;
    background: #ffffff;
    border-radius: 6px;
    box-shadow: 0 3px 6px rgba(0, 0, 0, 0.1);
  }

  .table td {
    display: flex;
    justify-content: space-between;
    padding: 12px;
    border-bottom: 1px solid #eee;
  }

  .table td:last-child {
    border-bottom: none;
  }

  .table td:first-child,
  .table td:nth-child(2) {
    min-width: 100%;
  }

  .table td::before {
    content: attr(data-label);
    font-weight: bold;
    color: #6007a5;
  }
}
</style>
