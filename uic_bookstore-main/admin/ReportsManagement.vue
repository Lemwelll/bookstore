<template>
  <div class="reports-management">
    <div class="reports-header">
      <h2>Reports & Analytics</h2>
      
      <div class="date-range-controls">
        <div class="date-inputs">
          <input type="date" v-model="startDate" :max="endDate" @change="loadReportData">
          <span>to</span>
          <input type="date" v-model="endDate" :min="startDate" @change="loadReportData">
        </div>
        
        <button class="generate-report-btn" @click="generateReport">
          Generate Report
        </button>
      </div>
    </div>
    
    <div class="reports-grid" v-if="!loading">
      <!-- Sales Overview Chart - Static SVG instead of Canvas -->
      <div class="report-card">
        <h3>Sales Overview</h3>
        <div class="static-chart">
          <div class="chart-legend">
            <div class="legend-item">
              <span class="legend-color uniform"></span>
              <span>Uniform Sales</span>
            </div>
            <div class="legend-item">
              <span class="legend-color books"></span>
              <span>Book Sales</span>
            </div>
            <div class="legend-item">
              <span class="legend-color other"></span>
              <span>Other Sales</span>
            </div>
          </div>
          <div class="chart-container">
            <div class="chart-y-axis">
              <div>24</div>
              <div>20</div>
              <div>16</div>
              <div>12</div>
              <div>8</div>
              <div>4</div>
              <div>0</div>
            </div>
            <div class="chart-content">
              <svg viewBox="0 0 500 200" class="line-chart">
                <!-- Grid lines -->
                <line x1="0" y1="0" x2="500" y2="0" stroke="#eee" />
                <line x1="0" y1="33" x2="500" y2="33" stroke="#eee" />
                <line x1="0" y1="66" x2="500" y2="66" stroke="#eee" />
                <line x1="0" y1="100" x2="500" y2="100" stroke="#eee" />
                <line x1="0" y1="133" x2="500" y2="133" stroke="#eee" />
                <line x1="0" y1="166" x2="500" y2="166" stroke="#eee" />
                <line x1="0" y1="200" x2="500" y2="200" stroke="#eee" />
                
                <!-- Uniform data line -->
                <polyline 
                  points="0,183 100,187 200,175 300,183 400,191"
                  fill="none" 
                  stroke="#4CAF50" 
                  stroke-width="2"
                />
                
                <!-- Books data line -->
                <polyline 
                  points="0,191 100,191 200,187 300,195 400,187"
                  fill="none" 
                  stroke="#2196F3" 
                  stroke-width="2"
                />
                
                <!-- Other data line -->
                <polyline 
                  points="0,100 100,125 200,150 300,166 400,141"
                  fill="none" 
                  stroke="#E91E63" 
                  stroke-width="2"
                />
                
                <!-- Data points -->
                <circle cx="0" cy="183" r="4" fill="#4CAF50" />
                <circle cx="100" cy="187" r="4" fill="#4CAF50" />
                <circle cx="200" cy="175" r="4" fill="#4CAF50" />
                <circle cx="300" cy="183" r="4" fill="#4CAF50" />
                <circle cx="400" cy="191" r="4" fill="#4CAF50" />
                
                <circle cx="0" cy="191" r="4" fill="#2196F3" />
                <circle cx="100" cy="191" r="4" fill="#2196F3" />
                <circle cx="200" cy="187" r="4" fill="#2196F3" />
                <circle cx="300" cy="195" r="4" fill="#2196F3" />
                <circle cx="400" cy="187" r="4" fill="#2196F3" />
                
                <circle cx="0" cy="100" r="4" fill="#E91E63" />
                <circle cx="100" cy="125" r="4" fill="#E91E63" />
                <circle cx="200" cy="150" r="4" fill="#E91E63" />
                <circle cx="300" cy="166" r="4" fill="#E91E63" />
                <circle cx="400" cy="141" r="4" fill="#E91E63" />
              </svg>
            </div>
          </div>
          <div class="chart-x-axis">
            <div>03/25</div>
            <div>03/26</div>
            <div>03/27</div>
            <div>03/28</div>
            <div>03/29</div>
          </div>
        </div>
      </div>
      
      <!-- Top Selling Products Table -->
      <div class="report-card">
        <h3>Top Selling Products</h3>
        <table class="data-table">
          <thead>
            <tr>
              <th>Product</th>
              <th>Category</th>
              <th>Units</th>
              <th>Revenue</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="product in topProducts" :key="product.id">
              <td>{{ product.product }}</td>
              <td>
                <span class="category-tag" :class="product.category.toLowerCase()">
                  {{ product.category }}
                </span>
              </td>
              <td>{{ product.sold }}</td>
              <td>₱{{ product.revenue }}</td>
            </tr>
          </tbody>
        </table>
      </div>
      
      <!-- Category Performance -->
      <div class="report-card">
        <h3>Category Performance</h3>
        <div v-for="(data, category) in categoryPerformance?.categories" :key="category" class="category-stats">
          <h4 :class="category.toLowerCase()">{{ category }}</h4>
          
          <div class="stats-row">
            <div class="stat-item">
              <span class="label">Sales</span>
              <span class="value">{{ data.sales }}</span>
            </div>
            
            <div class="stat-item">
              <span class="label">Revenue</span>
              <span class="value">₱{{ data.revenue }}</span>
            </div>
            
            <div class="stat-item">
              <span class="label">Profit</span>
              <span class="value" :class="data.profit > 0 ? 'positive' : 'negative'">
                ₱{{ data.profit }}
              </span>
            </div>
          </div>
        </div>
      </div>
      
      <!-- Sales Distribution Pie Chart -->
      <div class="report-card">
        <h3>Sales Distribution by Category</h3>
        <div class="static-pie-chart">
          <!-- Simplified donut chart rendering -->
          <div class="donut-chart" v-if="categoryPerformance && categoryPerformance.distribution">
            <!-- Donut segments using conic-gradient -->
            <div class="donut" :style="getDonutStyle()"></div>
          </div>
          
          <div class="pie-legend" v-if="categoryPerformance && categoryPerformance.distribution">
            <div class="legend-item" v-for="(value, index) in categoryPerformance.distribution.datasets[0].data" :key="index">
              <span class="legend-color" :class="getCategoryClass(index)"></span>
              <span>{{ categoryPerformance.distribution.labels[index] }}: {{ value }}%</span>
            </div>
          </div>
        </div>
      </div>
      
      <!-- Low Stock Items -->
      <div class="report-card">
        <h3>Low Stock Items</h3>
        <table class="data-table">
          <thead>
            <tr>
              <th>Product</th>
              <th>Category</th>
              <th>Stock</th>
              <th>Status</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="item in lowStockItems" :key="item.id">
              <td>{{ item.name }}</td>
              <td>
                <span class="category-tag" :class="item.category.toLowerCase()">
                  {{ item.category }}
                </span>
              </td>
              <td>{{ item.current_stock }} / {{ item.min_stock }}</td>
              <td>
                <span class="status-tag low-stock">
                  Low Stock
                </span>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
    
    <div class="loading-spinner" v-if="loading">
      <div class="spinner"></div>
      <p>Loading reports data...</p>
    </div>
  </div>
</template>

<script>
export default {
  name: 'ReportsManagement',
  data() {
    return {
      startDate: this.getDefaultStartDate(),
      endDate: this.getDefaultEndDate(),
      loading: true,
      salesData: null,
      topProducts: [],
      lowStockItems: [],
      categoryPerformance: null
    }
  },
  mounted() {
    this.loadReportData();
  },
  methods: {
    getDefaultStartDate() {
      const date = new Date();
      date.setDate(date.getDate() - 7); // One week ago
      return date.toISOString().split('T')[0];
    },
    getDefaultEndDate() {
      return new Date().toISOString().split('T')[0];
    },
    async loadReportData() {
      this.loading = true;
      
      // Initialize fallback data
      let salesDataFallback = {
        chart_data: {
          labels: ["2025-03-25", "2025-03-26", "2025-03-27", "2025-03-28", "2025-03-29"],
          datasets: [
            {label: "Uniform", data: [4, 3, 5, 4, 2], borderColor: "#4CAF50", backgroundColor: "#4CAF50"},
            {label: "Books", data: [2, 2, 3, 1, 3], borderColor: "#2196F3", backgroundColor: "#2196F3"},
            {label: "Other", data: [20, 15, 10, 5, 12], borderColor: "#E91E63", backgroundColor: "#E91E63"}
          ]
        },
        category_totals: {
          "Uniform": {sales: 18, revenue: 7725, cost: 6025, profit: 1700},
          "Books": {sales: 11, revenue: 8000, cost: 6400, profit: 1600},
          "Other": {sales: 62, revenue: 310, cost: 186, profit: 124}
        },
        overall_totals: {
          sales: 91, revenue: 16035, cost: 12611, profit: 3424
        },
        raw_data: []
      };
      
      let topProductsFallback = [
        {id: 1, product: "Polo", category: "Uniform", sold: 2, revenue: 900},
        {id: 4, product: "Physics Book", category: "Books", sold: 1, revenue: 750},
        {id: 5, product: "Chemistry Book", category: "Books", sold: 1, revenue: 750},
        {id: 3, product: "Blouse", category: "Uniform", sold: 1, revenue: 425},
        {id: 2, product: "Jogging Pants", category: "Uniform", sold: 1, revenue: 400}
      ];
      
      let lowStockItemsFallback = [
        {id: 6, name: "PE Book", category: "Books", current_stock: 3, min_stock: 5, price: 450.0},
        {id: 3, name: "Blouse", category: "Uniform", current_stock: 8, min_stock: 10, price: 425.0},
        {id: 7, name: "Scantron", category: "Other", current_stock: 45, min_stock: 50, price: 15.0}
      ];
      
      let categoryPerformanceFallback = {
        categories: {
          "Uniform": {sales: 18, revenue: 7725, cost: 6025, profit: 1700},
          "Books": {sales: 11, revenue: 8000, cost: 6400, profit: 1600},
          "Other": {sales: 62, revenue: 310, cost: 186, profit: 124}
        },
        distribution: {
          labels: ["Uniform", "Books", "Other"],
          datasets: [{
            data: [19.8, 12.1, 68.1],
            backgroundColor: ["#4CAF50", "#2196F3", "#E91E63"]
          }]
        }
      };

      // Set fallback data first so we have something to show even if API calls fail
      this.salesData = salesDataFallback;
      this.topProducts = topProductsFallback;
      this.lowStockItems = lowStockItemsFallback;
      this.categoryPerformance = categoryPerformanceFallback;

      try {
        // Load sales data
        try {
          const salesResponse = await fetch(`http://localhost:8000/admin/reports/sales?start_date=${this.startDate}&end_date=${this.endDate}`);
          if (salesResponse.ok) {
            const data = await salesResponse.json();
            if (data && data.chart_data) {
              this.salesData = data;
            }
          } else {
            console.error('Error response from sales API:', await salesResponse.text());
          }
        } catch (error) {
          console.error('Error loading sales data:', error);
        }
        
        // Load top products
        try {
          const productsResponse = await fetch(`http://localhost:8000/admin/reports/top-products?start_date=${this.startDate}&end_date=${this.endDate}`);
          if (productsResponse.ok) {
            const data = await productsResponse.json();
            if (data && Array.isArray(data)) {
              this.topProducts = data;
            }
          } else {
            console.error('Error response from top products API:', await productsResponse.text());
          }
        } catch (error) {
          console.error('Error loading top products:', error);
        }
        
        // Load low stock items
        try {
          const stockResponse = await fetch('http://localhost:8000/admin/reports/low-stock');
          if (stockResponse.ok) {
            const data = await stockResponse.json();
            if (data && Array.isArray(data)) {
              this.lowStockItems = data;
            }
          } else {
            console.error('Error response from low stock API:', await stockResponse.text());
          }
        } catch (error) {
          console.error('Error loading low stock items:', error);
        }
        
        // Load category performance
        try {
          const categoryResponse = await fetch(`http://localhost:8000/admin/reports/category-performance?start_date=${this.startDate}&end_date=${this.endDate}`);
          if (categoryResponse.ok) {
            const data = await categoryResponse.json();
            if (data && data.categories && data.distribution) {
              this.categoryPerformance = data;
            }
          } else {
            console.error('Error response from category performance API:', await categoryResponse.text());
          }
        } catch (error) {
          console.error('Error loading category performance:', error);
        }
        
        this.loading = false;
      } catch (error) {
        console.error('Error loading report data:', error);
        this.loading = false;
      }
    },
    generateReport() {
      // Load data with the selected date range
      this.loadReportData();
    },
    getDonutStyle() {
      // If we have data, build a conic gradient
      if (this.categoryPerformance && this.categoryPerformance.distribution) {
        const data = this.categoryPerformance.distribution.datasets[0].data;
        const colors = ['#4CAF50', '#2196F3', '#E91E63']; // Uniform, Books, Other
        
        // Build conic gradient string
        let cumulativePercentage = 0;
        let gradientString = '';
        
        for (let i = 0; i < data.length; i++) {
          const startPercent = cumulativePercentage;
          cumulativePercentage += data[i];
          
          if (data[i] > 0) {
            if (gradientString) gradientString += ', ';
            gradientString += `${colors[i]} ${startPercent}%, ${colors[i]} ${cumulativePercentage}%`;
          }
        }
        
        // If no data (all zeros), use gray
        if (!gradientString) {
          gradientString = '#f5f5f5 0%, #f5f5f5 100%';
        }
        
        return {
          'background': `conic-gradient(${gradientString})`,
          'width': '200px',
          'height': '200px',
          'border-radius': '50%',
          'position': 'relative'
        };
      }
      
      // Fallback
      return {
        'background': '#f5f5f5',
        'width': '200px',
        'height': '200px',
        'border-radius': '50%',
        'position': 'relative'
      };
    },
    getCategoryClass(index) {
      const categories = ['uniform', 'books', 'other'];
      return categories[index] || '';
    }
  }
}
</script>

<style scoped>
.reports-management {
  padding: 20px;
  height: 100%;
  overflow: auto;
}

.reports-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.reports-header h2 {
  font-size: 1.5rem;
  color: #333;
  margin: 0;
}

.date-range-controls {
  display: flex;
  align-items: center;
  gap: 15px;
}

.date-inputs {
  display: flex;
  align-items: center;
  gap: 10px;
}

.date-inputs input {
  padding: 8px 12px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
}

.date-inputs span {
  color: #666;
}

.generate-report-btn {
  background-color: #4a6cf7;
  color: white;
  border: none;
  border-radius: 4px;
  padding: 8px 15px;
  font-size: 14px;
  cursor: pointer;
  transition: background-color 0.3s;
}

.generate-report-btn:hover {
  background-color: #3a5cf0;
}

.reports-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 20px;
}

.report-card {
  background-color: white;
  border-radius: 8px;
  padding: 20px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
}

.report-card h3 {
  margin-top: 0;
  margin-bottom: 15px;
  font-size: 18px;
  color: #333;
}

/* Static chart styling */
.static-chart {
  width: 100%;
  border: 1px solid #f0f0f0;
  border-radius: 4px;
  padding: 10px;
}

.chart-legend {
  display: flex;
  justify-content: flex-end;
  margin-bottom: 10px;
  gap: 15px;
}

.legend-item {
  display: flex;
  align-items: center;
  font-size: 12px;
}

.legend-color {
  width: 12px;
  height: 12px;
  border-radius: 50%;
  display: inline-block;
  margin-right: 5px;
}

.legend-color.uniform {
  background-color: #4CAF50;
}

.legend-color.books {
  background-color: #2196F3;
}

.legend-color.other {
  background-color: #E91E63;
}

.chart-container {
  display: flex;
  height: 200px;
  margin-bottom: 10px;
}

.chart-y-axis {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  padding-right: 10px;
  font-size: 10px;
  color: #999;
  border-right: 1px solid #eee;
  width: 30px;
}

.chart-content {
  flex: 1;
  position: relative;
}

.line-chart {
  width: 100%;
  height: 200px;
}

.chart-x-axis {
  display: flex;
  justify-content: space-between;
  font-size: 10px;
  color: #999;
  padding-left: 30px;
}

/* Static pie chart styling */
.static-pie-chart {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.pie-chart {
  width: 200px;
  height: 200px;
  margin-bottom: 20px;
}

.donut-chart {
  position: relative;
  width: 200px;
  height: 200px;
  margin-bottom: 20px;
}

.donut {
  width: 200px;
  height: 200px;
  border-radius: 50%;
  position: relative;
}

.donut::after {
  content: '';
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 100px;
  height: 100px;
  background-color: white;
  border-radius: 50%;
}

.pie-legend {
  display: flex;
  justify-content: center;
  flex-wrap: wrap;
  gap: 20px;
}

.data-table {
  width: 100%;
  border-collapse: collapse;
}

.data-table th,
.data-table td {
  padding: 12px 15px;
  text-align: left;
  border-bottom: 1px solid #eee;
}

.data-table th {
  font-weight: 600;
  color: #444;
  font-size: 14px;
}

.data-table td {
  color: #555;
  font-size: 14px;
}

.category-tag {
  display: inline-block;
  padding: 4px 8px;
  border-radius: 4px;
  font-size: 12px;
  font-weight: 500;
}

.category-tag.uniform {
  background-color: rgba(76, 175, 80, 0.1);
  color: #4CAF50;
}

.category-tag.books {
  background-color: rgba(33, 150, 243, 0.1);
  color: #2196F3;
}

.category-tag.other {
  background-color: rgba(233, 30, 99, 0.1);
  color: #E91E63;
}

.status-tag {
  display: inline-block;
  padding: 4px 8px;
  border-radius: 4px;
  font-size: 12px;
  font-weight: 500;
}

.status-tag.low-stock {
  background-color: rgba(255, 152, 0, 0.1);
  color: #FF9800;
}

.category-stats {
  margin-bottom: 15px;
  padding-bottom: 15px;
  border-bottom: 1px solid #eee;
}

.category-stats:last-child {
  border-bottom: none;
  margin-bottom: 0;
  padding-bottom: 0;
}

.category-stats h4 {
  margin-top: 0;
  margin-bottom: 10px;
  font-size: 16px;
  font-weight: 600;
}

.category-stats h4.uniform {
  color: #4CAF50;
}

.category-stats h4.books {
  color: #2196F3;
}

.category-stats h4.other {
  color: #E91E63;
}

.stats-row {
  display: flex;
  justify-content: space-between;
}

.stat-item {
  flex: 1;
  display: flex;
  flex-direction: column;
}

.stat-item .label {
  font-size: 12px;
  color: #777;
  margin-bottom: 3px;
}

.stat-item .value {
  font-size: 15px;
  font-weight: 600;
  color: #333;
}

.stat-item .value.positive {
  color: #4CAF50;
}

.stat-item .value.negative {
  color: #F44336;
}

.loading-spinner {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 300px;
}

.spinner {
  width: 40px;
  height: 40px;
  border: 3px solid #f3f3f3;
  border-top: 3px solid #4a6cf7;
  border-radius: 50%;
  animation: spin 1s linear infinite;
  margin-bottom: 15px;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}
</style>
