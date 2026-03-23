<template>
  <v-btn
    @click="isActive = !isActive"
    class="menu-button ma-2"
    icon="mdi-menu">
  </v-btn>
  <div v-if="isActive" class="sidebar">
    <div class="sidebar-header">
      <span>Välj arter</span>
      <div class="sidebar-actions">
        <button @click="selectAll">Alla</button>
        <button @click="selectNone">Ingen</button>
        <button @click="collapseAll" title="Fäll ihop alla">▸▸</button>
      </div>
    </div>
    <div class="sidebar-list">
      <template v-if="groups && data">
        <div v-for="group in groups" :key="group.name" class="group">
          <!-- Group header row -->
          <div class="row group-header" @click="toggleExpand(group.name)">
            <input
              type="checkbox"
              :checked="groupAllSelected(group)"
              :indeterminate="groupPartialSelected(group)"
              @click.stop="toggleGroupSelection(group)"
            />
            <span class="row-name">{{ group.name }}</span>
            <span class="expand-arrow">{{ expanded[group.name] ? '▾' : '▸' }}</span>
          </div>
          <!-- Group body -->
          <div v-if="expanded[group.name]" class="group-body">
            <!-- Subcategories -->
            <template v-if="group.subcategories">
              <div v-for="sub in group.subcategories" :key="sub.name" class="subgroup">
                <div class="row subgroup-header" @click="toggleExpand(group.name + '/' + sub.name)">
                  <input
                    type="checkbox"
                    :checked="subAllSelected(sub)"
                    :indeterminate="subPartialSelected(sub)"
                    @click.stop="toggleSubSelection(sub)"
                  />
                  <span class="row-name">{{ sub.name }}</span>
                  <span class="expand-arrow">{{ expanded[group.name + '/' + sub.name] ? '▾' : '▸' }}</span>
                </div>
                <div v-if="expanded[group.name + '/' + sub.name]" class="species-list sub-species">
                  <div
                    v-for="id in sub.taxons"
                    :key="id"
                    class="row taxon-item"
                    @click="toggleTaxon(id)"
                  >
                    <input type="checkbox" :checked="selected[id]" @click.stop="toggleTaxon(id)" />
                    <span class="row-name">{{ data[id] }}</span>
                  </div>
                </div>
              </div>
            </template>
            <!-- Direct taxons -->
            <template v-else>
              <div class="species-list">
                <div
                  v-for="id in group.taxons"
                  :key="id"
                  class="row taxon-item"
                  @click="toggleTaxon(id)"
                >
                  <input type="checkbox" :checked="selected[id]" @click.stop="toggleTaxon(id)" />
                  <span class="row-name">{{ data[id] }}</span>
                </div>
              </div>
            </template>
          </div>
        </div>
      </template>
    </div>
  </div>
</template>

<script>
function groupTaxonIds(group) {
  if (group.subcategories) {
    return group.subcategories.flatMap(s => s.taxons);
  }
  return group.taxons || [];
}

export default {
  props: ['data', 'groups'],
  emits: ['update:selected'],
  data() {
    return {
      isActive: false,
      selected: {},
      expanded: {}
    };
  },
  watch: {
    data: {
      immediate: true,
      handler(newData) {
        if (!newData) return;
        const sel = {};
        Object.keys(newData).forEach(key => { sel[key] = true; });
        this.selected = sel;
        this.$emit('update:selected', { ...this.selected });
      }
    }
  },
  methods: {
    toggleExpand(key) {
      this.expanded = { ...this.expanded, [key]: !this.expanded[key] };
    },
    toggleTaxon(id) {
      this.selected = { ...this.selected, [id]: !this.selected[id] };
      this.$emit('update:selected', { ...this.selected });
    },
    toggleGroupSelection(group) {
      const ids = groupTaxonIds(group);
      const allOn = ids.every(id => this.selected[id]);
      const next = { ...this.selected };
      ids.forEach(id => { next[id] = !allOn; });
      this.selected = next;
      this.$emit('update:selected', { ...this.selected });
    },
    toggleSubSelection(sub) {
      const ids = sub.taxons || [];
      const allOn = ids.every(id => this.selected[id]);
      const next = { ...this.selected };
      ids.forEach(id => { next[id] = !allOn; });
      this.selected = next;
      this.$emit('update:selected', { ...this.selected });
    },
    groupAllSelected(group) {
      const ids = groupTaxonIds(group);
      return ids.length > 0 && ids.every(id => this.selected[id]);
    },
    groupPartialSelected(group) {
      const ids = groupTaxonIds(group);
      const count = ids.filter(id => this.selected[id]).length;
      return count > 0 && count < ids.length;
    },
    subAllSelected(sub) {
      const ids = sub.taxons || [];
      return ids.length > 0 && ids.every(id => this.selected[id]);
    },
    subPartialSelected(sub) {
      const ids = sub.taxons || [];
      const count = ids.filter(id => this.selected[id]).length;
      return count > 0 && count < ids.length;
    },
    collapseAll() {
      this.expanded = {};
    },
    selectAll() {
      const sel = {};
      Object.keys(this.data).forEach(key => { sel[key] = true; });
      this.selected = sel;
      this.$emit('update:selected', { ...this.selected });
    },
    selectNone() {
      const sel = {};
      Object.keys(this.data).forEach(key => { sel[key] = false; });
      this.selected = sel;
      this.$emit('update:selected', { ...this.selected });
    }
  }
};
</script>

<style scoped>
.menu-button {
  z-index: 999;
  position: fixed;
  top: 0;
  opacity: 0.5;
}

.sidebar {
  position: fixed;
  top: 0;
  left: 0;
  width: 240px;
  height: 100vh;
  background-color: #f1f1f1;
  z-index: 1000;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  box-shadow: 2px 0 6px rgba(0,0,0,0.15);
}

.sidebar-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 10px 12px;
  font-weight: bold;
  font-size: 0.95rem;
  border-bottom: 1px solid #ccc;
  flex-shrink: 0;
  background: #e4e4e4;
}

.sidebar-actions button {
  margin-left: 6px;
  font-size: 0.75rem;
  cursor: pointer;
  padding: 2px 6px;
  border: 1px solid #aaa;
  border-radius: 3px;
  background: #fff;
}

.sidebar-actions button:last-child {
  padding: 2px 4px;
  font-size: 0.65rem;
}

.sidebar-list {
  overflow-y: auto;
  flex: 1;
  font-size: 0.85rem;
}

.row {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 4px 10px;
  cursor: pointer;
  user-select: none;
}
.row:hover {
  background: #dde8ee;
}

.group-header {
  font-weight: 600;
  background: #e8e8e8;
  border-top: 1px solid #d0d0d0;
}

.subgroup-header {
  background: #f0f0f0;
  padding-left: 20px;
}

.row-name {
  flex: 1;
  text-transform: capitalize;
}

.expand-arrow {
  font-size: 0.75rem;
  color: #666;
}

.species-list {
  background: #f7f7f7;
}

.taxon-item {
  padding-left: 18px;
}

.sub-species .taxon-item {
  padding-left: 32px;
}

input[type="checkbox"] {
  flex-shrink: 0;
  cursor: pointer;
}
</style>