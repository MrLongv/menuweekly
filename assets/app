const DAYS = [
  { key: 'mon', label: 'Thứ 2' },
  { key: 'tue', label: 'Thứ 3' },
  { key: 'wed', label: 'Thứ 4' },
  { key: 'thu', label: 'Thứ 5' },
  { key: 'fri', label: 'Thứ 6' },
  { key: 'sat', label: 'Thứ 7' }
];

const state = {
  dishes: [],
  ingredients: [],
  settings: { mealPrice: 10000, defaultMeals: 1425 },
  recipesByDishId: {},
  weeklyMenu: null,
  activeTab: 'dashboard',
  editingDishId: null,
  editingIngredientId: null
};

const els = {};

document.addEventListener('DOMContentLoaded', () => {
  bindElements();
  initWeekStart();
  bindEvents();
  renderTabs();
  bootstrap();
});

function bindElements() {
  els.navButtons = document.querySelectorAll('.nav-link');
  els.tabPanels = document.querySelectorAll('.tab-panel');
  els.pageTitle = document.getElementById('pageTitle');
  els.pageSubtitle = document.getElementById('pageSubtitle');
  els.apiDot = document.getElementById('apiDot');
  els.apiStatusText = document.getElementById('apiStatusText');
  els.weekStartInput = document.getElementById('weekStartInput');
  els.reloadBtn = document.getElementById('reloadBtn');
  els.saveWeekBtn = document.getElementById('saveWeekBtn');
  els.menuGrid = document.getElementById('menuGrid');
  els.randomWeekBtn = document.getElementById('randomWeekBtn');
  els.duplicateLastWeekBtn = document.getElementById('duplicateLastWeekBtn');
  els.dishTableBody = document.getElementById('dishTableBody');
  els.dishForm = document.getElementById('dishForm');
  els.dishFormTitle = document.getElementById('dishFormTitle');
  els.resetDishFormBtn = document.getElementById('resetDishFormBtn');
  els.dishSearchInput = document.getElementById('dishSearchInput');
  els.dishFilterCategory = document.getElementById('dishFilterCategory');
  els.recipeDishSelect = document.getElementById('recipeDishSelect');
  els.recipeTableBody = document.getElementById('recipeTableBody');
  els.addRecipeRowBtn = document.getElementById('addRecipeRowBtn');
  els.saveRecipesBtn = document.getElementById('saveRecipesBtn');
  els.ingredientTableBody = document.getElementById('ingredientTableBody');
  els.ingredientForm = document.getElementById('ingredientForm');
  els.resetIngredientFormBtn = document.getElementById('resetIngredientFormBtn');
  els.ingredientSearchInput = document.getElementById('ingredientSearchInput');
  els.settingsForm = document.getElementById('settingsForm');
  els.settingApiBase = document.getElementById('settingApiBase');
  els.settingMealPrice = document.getElementById('settingMealPrice');
  els.settingDefaultMeals = document.getElementById('settingDefaultMeals');
  els.dailyReportCards = document.getElementById('dailyReportCards');
  els.weeklyReportPanel = document.getElementById('weeklyReportPanel');
  els.weekSummary = document.getElementById('weekSummary');
  els.toast = document.getElementById('toast');
}

function initWeekStart() {
  const now = new Date();
  const day = now.getDay();
  const diff = day === 0 ? -6 : 1 - day;
  now.setDate(now.getDate() + diff);
  els.weekStartInput.value = toDateInputValue(now);
}

function bindEvents() {
  els.navButtons.forEach((btn) => btn.addEventListener('click', () => switchTab(btn.dataset.tab)));
  els.reloadBtn.addEventListener('click', bootstrap);
  els.saveWeekBtn.addEventListener('click', saveWeeklyMenu);
  els.weekStartInput.addEventListener('change', loadWeekData);
  els.randomWeekBtn.addEventListener('click', randomizeWeek);
  els.duplicateLastWeekBtn.addEventListener('click', duplicatePreviousWeek);
  els.dishForm.addEventListener('submit', handleDishSubmit);
  els.resetDishFormBtn.addEventListener('click', resetDishForm);
  els.dishSearchInput.addEventListener('input', renderDishTable);
  els.dishFilterCategory.addEventListener('change', renderDishTable);
  els.recipeDishSelect.addEventListener('change', renderRecipeTable);
  els.addRecipeRowBtn.addEventListener('click', () => addRecipeRow());
  els.saveRecipesBtn.addEventListener('click', saveRecipes);
  els.ingredientForm.addEventListener('submit', handleIngredientSubmit);
  els.resetIngredientFormBtn.addEventListener('click', resetIngredientForm);
  els.ingredientSearchInput.addEventListener('input', renderIngredientTable);
  els.settingsForm.addEventListener('submit', saveSettings);
}

async function bootstrap() {
  els.settingApiBase.value = window.APP_CONFIG.API_BASE;
  const ok = await pingApi();
  if (!ok) return;
  await Promise.all([loadBaseData(), loadWeekData()]);
}

async function pingApi() {
  try {
    const response = await fetch(`${window.APP_CONFIG.API_BASE}/api/health`);
    const data = await response.json();
    els.apiDot.className = 'status-dot ok';
    els.apiStatusText.textContent = data.message || 'Kết nối tốt';
    return true;
  } catch (error) {
    console.error(error);
    els.apiDot.className = 'status-dot fail';
    els.apiStatusText.textContent = 'Không kết nối được Worker';
    showToast('Chưa kết nối được Cloudflare Worker');
    return false;
  }
}

async function loadBaseData() {
  const [dishes, ingredients, settings] = await Promise.all([
    api('/api/dishes'),
    api('/api/ingredients'),
    api('/api/settings')
  ]);

  state.dishes = dishes.items || [];
  state.ingredients = ingredients.items || [];
  state.settings = settings.item || state.settings;

  renderDishTable();
  renderIngredientTable();
  renderRecipeDishOptions();
  renderSettings();
}

async function loadWeekData() {
  const weekStart = els.weekStartInput.value;
  const payload = await api(`/api/menu?weekStart=${encodeURIComponent(weekStart)}`);
  state.weeklyMenu = normalizeWeeklyMenu(payload.item, weekStart, state.settings.defaultMeals, state.settings.mealPrice);
  renderMenuGrid();
  await renderReports();
  renderDashboard();
}

function normalizeWeeklyMenu(item, weekStart, defaultMeals, mealPrice) {
  if (item?.days?.length) return item;
  return {
    id: null,
    weekStart,
    days: DAYS.map((day, index) => ({
      dayNo: index + 1,
      dayKey: day.key,
      label: day.label,
      workerMeals: defaultMeals,
      guestMeals: 0,
      targetPricePerMeal: mealPrice,
      dishManId: '',
      dishXaoId: '',
      dishCanhId: ''
    }))
  };
}

function renderTabs() {
  const titles = {
    dashboard: ['Tổng quan', 'Theo dõi thực đơn, công thức và cân đối chi phí theo tuần.'],
    menu: ['Thực đơn tuần', 'Lập menu tuần và lưu về Cloudflare D1.'],
    dishes: ['Thư viện món', 'Quản lý món ăn dùng trong thực đơn.'],
    recipes: ['Công thức món', 'Định mức nguyên liệu cho từng món.'],
    ingredients: ['Nguyên liệu & giá', 'Kho nguyên liệu, gia vị và đơn giá.'],
    reports: ['Cân đối tuần', 'Chi phí, suất ăn bình quân, thừa thiếu, lũy kế.'],
    settings: ['Cài đặt', 'Thông số mặc định cho toàn hệ thống.']
  };
  const [title, subtitle] = titles[state.activeTab];
  els.pageTitle.textContent = title;
  els.pageSubtitle.textContent = subtitle;
  els.navButtons.forEach((btn) => btn.classList.toggle('active', btn.dataset.tab === state.activeTab));
  els.tabPanels.forEach((panel) => panel.classList.toggle('active', panel.id === `tab-${state.activeTab}`));
}

function switchTab(tab) {
  state.activeTab = tab;
  renderTabs();
}

function renderMenuGrid() {
  const optionsMan = getDishOptions('man');
  const optionsXao = getDishOptions('xao');
  const optionsCanh = getDishOptions('canh');

  els.menuGrid.innerHTML = state.weeklyMenu.days.map((day, index) => `
    <article class="menu-day">
      <h4>
        <span>${day.label}</span>
        <button class="small-btn" data-random-day="${index}">🎲 Ngày này</button>
      </h4>
      <div class="day-meta">
        <label class="field">
          <span>Suất CN</span>
          <input type="number" min="0" value="${day.workerMeals}" data-field="workerMeals" data-index="${index}" />
        </label>
        <label class="field">
          <span>Khách</span>
          <input type="number" min="0" value="${day.guestMeals}" data-field="guestMeals" data-index="${index}" />
        </label>
        <label class="field">
          <span>Định mức / suất</span>
          <input type="number" min="0" value="${day.targetPricePerMeal}" data-field="targetPricePerMeal" data-index="${index}" />
        </label>
      </div>
      <div class="day-selects">
        <label class="field">
          <span>Món mặn</span>
          <select data-field="dishManId" data-index="${index}">${renderOptionMarkup(optionsMan, day.dishManId)}</select>
        </label>
        <label class="field">
          <span>Món xào</span>
          <select data-field="dishXaoId" data-index="${index}">${renderOptionMarkup(optionsXao, day.dishXaoId)}</select>
        </label>
        <label class="field">
          <span>Món canh</span>
          <select data-field="dishCanhId" data-index="${index}">${renderOptionMarkup(optionsCanh, day.dishCanhId)}</select>
        </label>
      </div>
    </article>
  `).join('');

  els.menuGrid.querySelectorAll('[data-field]').forEach((node) => {
    node.addEventListener('change', handleMenuFieldChange);
    node.addEventListener('input', handleMenuFieldChange);
  });

  els.menuGrid.querySelectorAll('[data-random-day]').forEach((btn) => btn.addEventListener('click', () => {
    randomizeDay(Number(btn.dataset.randomDay));
  }));
}

function handleMenuFieldChange(event) {
  const index = Number(event.target.dataset.index);
  const field = event.target.dataset.field;
  const value = event.target.type === 'number' ? Number(event.target.value || 0) : event.target.value;
  state.weeklyMenu.days[index][field] = value;
  renderDashboard();
  renderReports();
}

function getDishOptions(category) {
  return state.dishes.filter((dish) => dish.category === category && dish.active !== 0);
}

function renderOptionMarkup(items, selectedId) {
  return [`<option value="">-- Chọn món --</option>`].concat(
    items.map((item) => `<option value="${item.id}" ${String(item.id) === String(selectedId) ? 'selected' : ''}>${escapeHtml(item.name)}</option>`)
  ).join('');
}

async function saveWeeklyMenu() {
  try {
    const payload = { weekStart: state.weeklyMenu.weekStart, days: state.weeklyMenu.days };
    const result = await api('/api/menu', { method: 'POST', body: JSON.stringify(payload) });
    state.weeklyMenu.id = result.item?.id || state.weeklyMenu.id;
    showToast('Đã lưu thực đơn tuần');
    await renderReports();
  } catch (error) {
    console.error(error);
    showToast('Lưu tuần thất bại');
  }
}

function randomizeWeek() {
  state.weeklyMenu.days.forEach((_, index) => randomizeDay(index, false));
  renderMenuGrid();
  renderDashboard();
  renderReports();
}

function randomizeDay(index, rerender = true) {
  const day = state.weeklyMenu.days[index];
  day.dishManId = pickRandomId(getDishOptions('man'));
  day.dishXaoId = pickRandomId(getDishOptions('xao'));
  day.dishCanhId = pickRandomId(getDishOptions('canh'));
  if (rerender) {
    renderMenuGrid();
    renderDashboard();
    renderReports();
  }
}

async function duplicatePreviousWeek() {
  try {
    const data = await api(`/api/menu/previous?weekStart=${encodeURIComponent(state.weeklyMenu.weekStart)}`);
    if (!data.item?.days?.length) {
      showToast('Chưa có tuần trước để sao chép');
      return;
    }
    state.weeklyMenu = normalizeWeeklyMenu(data.item, state.weeklyMenu.weekStart, state.settings.defaultMeals, state.settings.mealPrice);
    state.weeklyMenu.weekStart = els.weekStartInput.value;
    renderMenuGrid();
    renderDashboard();
    renderReports();
    showToast('Đã dùng dữ liệu tuần gần nhất');
  } catch (error) {
    console.error(error);
    showToast('Không tải được tuần gần nhất');
  }
}

function renderDishTable() {
  const keyword = els.dishSearchInput.value.trim().toLowerCase();
  const category = els.dishFilterCategory.value;
  const rows = state.dishes.filter((dish) => {
    const matchedKeyword = !keyword || dish.name.toLowerCase().includes(keyword);
    const matchedCategory = category === 'all' || dish.category === category;
    return matchedKeyword && matchedCategory;
  });

  els.dishTableBody.innerHTML = rows.map((dish) => `
    <tr>
      <td>${escapeHtml(dish.name)}</td>
      <td>${categoryLabel(dish.category)}</td>
      <td>${dish.active !== 0 ? '<span class="badge">Đang dùng</span>' : '<span class="badge">Tạm ẩn</span>'}</td>
      <td>${escapeHtml(dish.notes || '')}</td>
      <td>
        <button class="small-btn" data-edit-dish="${dish.id}">Sửa</button>
        <button class="small-btn danger" data-delete-dish="${dish.id}">Xóa</button>
      </td>
    </tr>
  `).join('');

  els.dishTableBody.querySelectorAll('[data-edit-dish]').forEach((btn) => btn.addEventListener('click', () => fillDishForm(btn.dataset.editDish)));
  els.dishTableBody.querySelectorAll('[data-delete-dish]').forEach((btn) => btn.addEventListener('click', () => deleteDish(btn.dataset.deleteDish)));
  document.getElementById('statDishCount').textContent = state.dishes.filter((dish) => dish.active !== 0).length;
}

function fillDishForm(id) {
  const dish = state.dishes.find((item) => String(item.id) === String(id));
  if (!dish) return;
  state.editingDishId = id;
  els.dishFormTitle.textContent = 'Sửa món';
  document.getElementById('dishId').value = dish.id;
  document.getElementById('dishName').value = dish.name;
  document.getElementById('dishCategory').value = dish.category;
  document.getElementById('dishNotes').value = dish.notes || '';
  document.getElementById('dishActive').checked = dish.active !== 0;
}

function resetDishForm() {
  state.editingDishId = null;
  els.dishFormTitle.textContent = 'Thêm món';
  els.dishForm.reset();
  document.getElementById('dishActive').checked = true;
}

async function handleDishSubmit(event) {
  event.preventDefault();
  const payload = {
    id: document.getElementById('dishId').value || undefined,
    name: document.getElementById('dishName').value.trim(),
    category: document.getElementById('dishCategory').value,
    notes: document.getElementById('dishNotes').value.trim(),
    active: document.getElementById('dishActive').checked ? 1 : 0
  };

  try {
    await api('/api/dishes', { method: 'POST', body: JSON.stringify(payload) });
    await loadBaseData();
    resetDishForm();
    renderMenuGrid();
    showToast('Đã lưu món');
  } catch (error) {
    console.error(error);
    showToast('Lưu món thất bại');
  }
}

async function deleteDish(id) {
  if (!confirm('Xóa món này?')) return;
  try {
    await api(`/api/dishes/${id}`, { method: 'DELETE' });
    await loadBaseData();
    renderMenuGrid();
    showToast('Đã xóa món');
  } catch (error) {
    console.error(error);
    showToast('Xóa món thất bại');
  }
}

function renderRecipeDishOptions() {
  const currentId = els.recipeDishSelect.value;
  els.recipeDishSelect.innerHTML = state.dishes.map((dish) => `<option value="${dish.id}">${escapeHtml(dish.name)} (${categoryLabel(dish.category)})</option>`).join('');
  if (currentId) els.recipeDishSelect.value = currentId;
  if (!els.recipeDishSelect.value && state.dishes[0]) els.recipeDishSelect.value = state.dishes[0].id;
  renderRecipeTable();
}

async function renderRecipeTable() {
  const dishId = els.recipeDishSelect.value;
  if (!dishId) {
    els.recipeTableBody.innerHTML = '';
    return;
  }
  if (!state.recipesByDishId[dishId]) {
    const response = await api(`/api/recipes?dishId=${encodeURIComponent(dishId)}`);
    state.recipesByDishId[dishId] = response.items || [];
  }
  const rows = state.recipesByDishId[dishId];
  els.recipeTableBody.innerHTML = rows.map((row, index) => renderRecipeRow(row, index)).join('');
  bindRecipeRowEvents();
}

function addRecipeRow() {
  const dishId = els.recipeDishSelect.value;
  if (!dishId) return;
  state.recipesByDishId[dishId] = state.recipesByDishId[dishId] || [];
  state.recipesByDishId[dishId].push({ ingredientId: '', unit: '', qtyPer100: 0, costGroup: 'market', notes: '' });
  renderRecipeTable();
}

function renderRecipeRow(row, index) {
  const ingredientOptions = [`<option value="">-- chọn --</option>`].concat(
    state.ingredients.map((item) => `<option value="${item.id}" ${String(item.id) === String(row.ingredientId) ? 'selected' : ''}>${escapeHtml(item.name)}</option>`)
  ).join('');
  return `
    <tr>
      <td><select data-recipe-index="${index}" data-recipe-field="ingredientId">${ingredientOptions}</select></td>
      <td><input data-recipe-index="${index}" data-recipe-field="unit" value="${escapeHtml(row.unit || '')}" /></td>
      <td><input type="number" min="0" step="0.01" data-recipe-index="${index}" data-recipe-field="qtyPer100" value="${Number(row.qtyPer100 || 0)}" /></td>
      <td>
        <select data-recipe-index="${index}" data-recipe-field="costGroup">
          <option value="market" ${row.costGroup === 'market' ? 'selected' : ''}>Tiền chợ</option>
          <option value="warehouse" ${row.costGroup === 'warehouse' ? 'selected' : ''}>Gia vị xuất kho</option>
          <option value="fixed" ${row.costGroup === 'fixed' ? 'selected' : ''}>Cố định</option>
        </select>
      </td>
      <td><input data-recipe-index="${index}" data-recipe-field="notes" value="${escapeHtml(row.notes || '')}" /></td>
      <td><button class="small-btn danger" data-recipe-remove="${index}">Xóa</button></td>
    </tr>
  `;
}

function bindRecipeRowEvents() {
  els.recipeTableBody.querySelectorAll('[data-recipe-field]').forEach((node) => node.addEventListener('change', handleRecipeFieldChange));
  els.recipeTableBody.querySelectorAll('[data-recipe-remove]').forEach((btn) => btn.addEventListener('click', () => {
    const dishId = els.recipeDishSelect.value;
    state.recipesByDishId[dishId].splice(Number(btn.dataset.recipeRemove), 1);
    renderRecipeTable();
  }));
}

function handleRecipeFieldChange(event) {
  const dishId = els.recipeDishSelect.value;
  const rows = state.recipesByDishId[dishId] || [];
  const index = Number(event.target.dataset.recipeIndex);
  const field = event.target.dataset.recipeField;
  rows[index][field] = event.target.type === 'number' ? Number(event.target.value || 0) : event.target.value;
}

async function saveRecipes() {
  const dishId = els.recipeDishSelect.value;
  try {
    await api('/api/recipes', {
      method: 'POST',
      body: JSON.stringify({ dishId, items: state.recipesByDishId[dishId] || [] })
    });
    showToast('Đã lưu công thức');
    await renderReports();
  } catch (error) {
    console.error(error);
    showToast('Lưu công thức thất bại');
  }
}

function renderIngredientTable() {
  const keyword = els.ingredientSearchInput.value.trim().toLowerCase();
  const rows = state.ingredients.filter((item) => !keyword || item.name.toLowerCase().includes(keyword));
  els.ingredientTableBody.innerHTML = rows.map((item) => `
    <tr>
      <td>${escapeHtml(item.name)}</td>
      <td>${escapeHtml(item.unit)}</td>
      <td>${formatCurrency(item.defaultPrice || 0)}</td>
      <td>${costGroupLabel(item.costGroup)}</td>
      <td>
        <button class="small-btn" data-edit-ingredient="${item.id}">Sửa</button>
        <button class="small-btn danger" data-delete-ingredient="${item.id}">Xóa</button>
      </td>
    </tr>
  `).join('');

  els.ingredientTableBody.querySelectorAll('[data-edit-ingredient]').forEach((btn) => btn.addEventListener('click', () => fillIngredientForm(btn.dataset.editIngredient)));
  els.ingredientTableBody.querySelectorAll('[data-delete-ingredient]').forEach((btn) => btn.addEventListener('click', () => deleteIngredient(btn.dataset.deleteIngredient)));
  document.getElementById('statIngredientCount').textContent = state.ingredients.length;
}

function fillIngredientForm(id) {
  const item = state.ingredients.find((row) => String(row.id) === String(id));
  if (!item) return;
  state.editingIngredientId = id;
  document.getElementById('ingredientFormTitle').textContent = 'Sửa nguyên liệu';
  document.getElementById('ingredientId').value = item.id;
  document.getElementById('ingredientName').value = item.name;
  document.getElementById('ingredientUnit').value = item.unit;
  document.getElementById('ingredientPrice').value = item.defaultPrice || 0;
  document.getElementById('ingredientCostGroup').value = item.costGroup;
}

function resetIngredientForm() {
  state.editingIngredientId = null;
  document.getElementById('ingredientFormTitle').textContent = 'Thêm nguyên liệu';
  els.ingredientForm.reset();
}

async function handleIngredientSubmit(event) {
  event.preventDefault();
  const payload = {
    id: document.getElementById('ingredientId').value || undefined,
    name: document.getElementById('ingredientName').value.trim(),
    unit: document.getElementById('ingredientUnit').value.trim(),
    defaultPrice: Number(document.getElementById('ingredientPrice').value || 0),
    costGroup: document.getElementById('ingredientCostGroup').value
  };
  try {
    await api('/api/ingredients', { method: 'POST', body: JSON.stringify(payload) });
    await loadBaseData();
    resetIngredientForm();
    showToast('Đã lưu nguyên liệu');
  } catch (error) {
    console.error(error);
    showToast('Lưu nguyên liệu thất bại');
  }
}

async function deleteIngredient(id) {
  if (!confirm('Xóa nguyên liệu này?')) return;
  try {
    await api(`/api/ingredients/${id}`, { method: 'DELETE' });
    await loadBaseData();
    showToast('Đã xóa nguyên liệu');
  } catch (error) {
    console.error(error);
    showToast('Xóa nguyên liệu thất bại');
  }
}

function renderSettings() {
  els.settingMealPrice.value = state.settings.mealPrice ?? 10000;
  els.settingDefaultMeals.value = state.settings.defaultMeals ?? 1425;
}

async function saveSettings(event) {
  event.preventDefault();
  try {
    const payload = {
      mealPrice: Number(els.settingMealPrice.value || 0),
      defaultMeals: Number(els.settingDefaultMeals.value || 0)
    };
    await api('/api/settings', { method: 'POST', body: JSON.stringify(payload) });
    state.settings = payload;
    showToast('Đã lưu cài đặt');
    await loadWeekData();
  } catch (error) {
    console.error(error);
    showToast('Lưu cài đặt thất bại');
  }
}

async function renderReports() {
  const payload = await api('/api/report/weekly', {
    method: 'POST',
    body: JSON.stringify({ weekStart: state.weeklyMenu.weekStart, days: state.weeklyMenu.days })
  });

  const report = payload.item || { days: [], totals: {} };
  els.dailyReportCards.innerHTML = report.days.map((day) => `
    <article class="report-card">
      <strong>${day.label}</strong>
      <div class="summary-list">
        <div class="summary-item">Tiền chợ: <strong>${formatCurrency(day.marketCost)}</strong></div>
        <div class="summary-item">Gia vị / kho: <strong>${formatCurrency(day.warehouseCost)}</strong></div>
        <div class="summary-item">Tổng chi: <strong>${formatCurrency(day.totalCost)}</strong></div>
        <div class="summary-item">Bình quân suất: <strong>${formatCurrency(day.avgCostPerMeal)}</strong></div>
        <div class="summary-item">Thừa / thiếu: <strong>${formatCurrency(day.balance)}</strong></div>
      </div>
    </article>
  `).join('');

  els.weeklyReportPanel.innerHTML = `
    <div class="summary-item">Tổng tiền chợ: <strong>${formatCurrency(report.totals.marketCost || 0)}</strong></div>
    <div class="summary-item">Tổng gia vị / kho: <strong>${formatCurrency(report.totals.warehouseCost || 0)}</strong></div>
    <div class="summary-item">Tổng chi phí tuần: <strong>${formatCurrency(report.totals.totalCost || 0)}</strong></div>
    <div class="summary-item">Quỹ định mức tuần: <strong>${formatCurrency(report.totals.budget || 0)}</strong></div>
    <div class="summary-item">Thừa / thiếu tuần: <strong>${formatCurrency(report.totals.balance || 0)}</strong></div>
  `;

  document.getElementById('statWeekCost').textContent = formatCurrency(report.totals.totalCost || 0);
  document.getElementById('statWeekBalance').textContent = formatCurrency(report.totals.balance || 0);

  els.weekSummary.innerHTML = report.days.map((day) => `
    <div class="summary-item">
      <strong>${day.label}</strong>
      <div class="muted">${dishName(day.dishManId)} • ${dishName(day.dishXaoId)} • ${dishName(day.dishCanhId)}</div>
      <div class="muted">${day.totalMeals} suất · Tổng chi ${formatCurrency(day.totalCost)}</div>
    </div>
  `).join('');
}

function renderDashboard() {
  // số liệu thực được cập nhật ở renderReports
}

function pickRandomId(items) {
  if (!items.length) return '';
  const index = Math.floor(Math.random() * items.length);
  return items[index].id;
}

async function api(path, options = {}) {
  const response = await fetch(`${window.APP_CONFIG.API_BASE}${path}`, {
    headers: { 'Content-Type': 'application/json', ...(options.headers || {}) },
    ...options
  });
  if (!response.ok) {
    const text = await response.text();
    throw new Error(text || 'API request failed');
  }
  return response.json();
}

function toDateInputValue(date) {
  const year = date.getFullYear();
  const month = String(date.getMonth() + 1).padStart(2, '0');
  const day = String(date.getDate()).padStart(2, '0');
  return `${year}-${month}-${day}`;
}

function formatCurrency(value) {
  return new Intl.NumberFormat('vi-VN', { style: 'currency', currency: 'VND', maximumFractionDigits: 0 }).format(Number(value || 0));
}

function categoryLabel(value) {
  return ({ man: 'Món mặn', xao: 'Món xào', canh: 'Món canh' }[value]) || value || '';
}

function costGroupLabel(value) {
  return ({ market: 'Tiền chợ', warehouse: 'Gia vị xuất kho', fixed: 'Chi phí cố định' }[value]) || value || '';
}

function dishName(id) {
  return state.dishes.find((dish) => String(dish.id) === String(id))?.name || 'Chưa chọn';
}

function showToast(message) {
  els.toast.textContent = message;
  els.toast.classList.add('show');
  clearTimeout(showToast._timer);
  showToast._timer = setTimeout(() => els.toast.classList.remove('show'), 2400);
}

function escapeHtml(value) {
  return String(value ?? '')
    .replaceAll('&', '&amp;')
    .replaceAll('<', '&lt;')
    .replaceAll('>', '&gt;')
    .replaceAll('"', '&quot;')
    .replaceAll("'", '&#39;');
}
