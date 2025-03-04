4. Implement line-bar-chart.component.ts
Modify line-bar-chart.component.ts:

typescript
Copy
Edit
import { Component, Input, OnChanges } from '@angular/core';
import * as Highcharts from 'highcharts';

@Component({
  selector: 'app-line-bar-chart',
  standalone: true,
  templateUrl: './line-bar-chart.component.html',
  styleUrls: ['./line-bar-chart.component.css']
})
export class LineBarChartComponent implements OnChanges {
  @Input() chartData!: any;  // Data from parent
  Highcharts = Highcharts;
  chartOptions: any;

  ngOnChanges() {
    if (!this.chartData || this.chartData.length === 0) return;

    // Convert date format for X-axis
    const categories = this.chartData.map((item: any) =>
      new Date(item.date).toLocaleDateString('en-GB', { day: '2-digit', month: 'short', year: 'numeric' })
    );

    // Prepare data series
    const activeOrders = this.chartData.map((item: any) => item.numActiveOrders);
    const inactiveOrders = this.chartData.map((item: any) => item.numInactiveOrders);

    this.chartOptions = {
      chart: { zoomType: 'xy' },
      title: { text: 'Active vs Inactive Orders' },
      xAxis: { categories },
      yAxis: [
        { title: { text: 'Active Orders' } },
        { title: { text: 'Inactive Orders' }, opposite: true }
      ],
      series: [
        { name: 'Active Orders', type: 'column', data: activeOrders },
        { name: 'Inactive Orders', type: 'line', data: inactiveOrders, yAxis: 1 }
      ]
    };
  }
}
5. Implement line-bar-chart.component.html
Modify line-bar-chart.component.html:

html
Copy
Edit
<div *ngIf="chartOptions">
  <highcharts-chart
    [Highcharts]="Highcharts"
    [options]="chartOptions"
    style="width: 100%; height: 400px; display: block;">
  </highcharts-chart>
</div>
6. Modify Parent Component (app.component.ts)
Modify app.component.ts to:

Load JSON data.
Implement filtering logic for This Week, This Month, Last 6 Months.
typescript
Copy
Edit
import { Component, OnInit } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { CommonModule } from '@angular/common';
import { LineBarChartComponent } from './components/line-bar-chart/line-bar-chart.component';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [CommonModule, LineBarChartComponent],
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent implements OnInit {
  selectedFilter = 'thisWeek';
  orderData: any[] = [];
  filteredData: any[] = [];

  constructor(private http: HttpClient) {}

  ngOnInit() {
    this.loadOrderData();
  }

  loadOrderData() {
    this.http.get<any[]>('assets/orders.json').subscribe(data => {
      this.orderData = data.map(item => ({
        date: new Date(item.date),
        numActiveOrders: item.numActiveOrders,
        numInactiveOrders: item.numInactiveOrders
      }));
      this.updateFilteredData();
    });
  }

  updateFilteredData() {
    const now = new Date();
    this.filteredData = this.orderData.filter(item => {
      const orderDate = new Date(item.date);
      switch (this.selectedFilter) {
        case 'thisWeek':
          const startOfWeek = new Date(now);
          startOfWeek.setDate(now.getDate() - now.getDay()); // Start of week (Sunday)
          return orderDate >= startOfWeek;
        case 'thisMonth':
          return orderDate.getMonth() === now.getMonth() && orderDate.getFullYear() === now.getFullYear();
        case 'last6Months':
          const sixMonthsAgo = new Date(now);
          sixMonthsAgo.setMonth(now.getMonth() - 6);
          return orderDate >= sixMonthsAgo;
        default:
          return true;
      }
    });
  }

  onFilterChange(filter: string) {
    this.selectedFilter = filter;
    this.updateFilteredData();
  }
}
7. Modify Parent Template (app.component.html)
Modify app.component.html to format the filter and chart inside a box:

html
Copy
Edit
<div class="chart-container">
  <div class="filter-box">
    <label for="filter">Filter: </label>
    <select id="filter" (change)="onFilterChange($event.target.value)">
      <option value="thisWeek">This Week</option>
      <option value="thisMonth">This Month</option>
      <option value="last6Months">Last 6 Months</option>
    </select>
  </div>

  <div class="chart-box">
    <app-line-bar-chart [chartData]="filteredData"></app-line-bar-chart>
  </div>
</div>
8. Apply Styles (app.component.css)
Modify app.component.css for better UI:

css
Copy
Edit
.chart-container {
  width: 80%;
  margin: auto;
  padding: 20px;
  border-radius: 10px;
  background: #fff;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  display: flex;
  flex-direction: column;
  align-items: center;
}

.filter-box {
  width: 100%;
  text-align: center;
  margin-bottom: 15px;
}

.filter-box label {
  font-weight: bold;
  margin-right: 10px;
}

.filter-box select {
  padding: 8px;
  border-radius: 5px;
  border: 1px solid #ccc;
  font-size: 14px;
}

.chart-box {
  width: 100%;
}
 
