<!DOCTYPE html>
<html lang="en" x-data="dealerDashboard" x-init="init" :class="theme">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Dealer Loyalty Dashboard</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <link href="https://cdnjs.cloudflare.com/ajax/libs/flowbite/1.6.6/flowbite.min.css" rel="stylesheet" />
  <script defer src="https://unpkg.com/alpinejs@3.x.x/dist/cdn.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
</head>
<body class="flex bg-gray-100 dark:bg-gray-900 text-gray-800 dark:text-gray-200 min-h-screen transition-colors">
  <aside class="fixed inset-y-0 left-0 w-60 bg-white dark:bg-gray-800 border-r dark:border-gray-700 p-4">
    <h2 class="text-2xl font-bold mb-6">Loyalty Panel</h2>
    <nav class="space-y-2">
      <button @click="tab='dealer'" :class="tab==='dealer' ? 'bg-indigo-500 text-white' : 'text-gray-600 dark:text-gray-300'" class="w-full text-left px-4 py-2 rounded-lg hover:bg-indigo-400 hover:text-white transition">
        Dealer View
      </button>
    </nav>
  </aside>
  <div class="flex-1 ml-60 p-6">
    <section x-show="tab==='dealer'" x-cloak class="space-y-8">
      <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-4 mb-6">
        <button @click="showModal = 'daily'" class="bg-white dark:bg-gray-700 p-4 rounded-lg shadow border border-gray-200 dark:border-gray-600 text-center">
          <h4 class="font-semibold mb-2">Daily Commissions</h4>
          <p class="text-xl font-bold text-green-600 dark:text-green-400" x-text="'$' + dailyCommissions.toFixed(2)"></p>
        </button>
        <button @click="showModal = 'awards'" class="bg-white dark:bg-gray-700 p-4 rounded-lg shadow border border-gray-200 dark:border-gray-600 text-center">
          <h4 class="font-semibold mb-2">Total Awards</h4>
          <p class="text-xl font-bold text-yellow-600 dark:text-yellow-400" x-text="'$' + totalAwards.toFixed(2)"></p>
        </button>
        <button @click="showModal = 'ranking'" class="bg-white dark:bg-gray-700 p-4 rounded-lg shadow border border-gray-200 dark:border-gray-600 text-center">
          <h4 class="font-semibold mb-2">Your Rank</h4>
          <p class="text-xl font-bold" x-text="userRank"></p>
        </button>
      </div>

      <template x-if="showModal">
        <div class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
          <div class="bg-white dark:bg-gray-800 p-6 rounded shadow w-96 max-h-[80vh] overflow-y-auto">
            <button class="text-red-500 float-right" @click="showModal = null">✖</button>

            <div x-show="showModal === 'daily'">
              <h2 class="text-lg font-bold mb-2">Daily Commissions</h2>
              <ul>
                <template x-for="commission in dailyCommissionList" :key="commission">
                  <li x-text="commission" class="mb-1"></li>
                </template>
              </ul>
            </div>

            <div x-show="showModal === 'awards'">
              <h2 class="text-lg font-bold mb-2">Awards Received</h2>
              <ul>
                <template x-for="award in awardList" :key="award">
                  <li x-text="award" class="mb-1"></li>
                </template>
              </ul>
            </div>

            <div x-show="showModal === 'ranking'">
              <h2 class="text-lg font-bold mb-2">Top 10 Dealers</h2>
			  <h2 class="text-lg font-bold mb-2">Name - Total Sales Amount($)</h2>
              <ul>
                <template x-for="(dealer, i) in topDealers" :key="dealer.name">
                  <li><span x-text="(i + 1) + '. ' + dealer.name + ' - ' + dealer.sales"></span></li>
                </template>
              </ul>
            </div>

          </div>
        </div>
      </template>

      <div class="card">
        <h3 class="mb-4 text-xl font-semibold">Incentives</h3>
        <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-4">
          <template x-for="incentive in incentives" :key="incentive.id">
            <div class="bg-white dark:bg-gray-700 p-4 rounded-lg shadow border border-gray-200 dark:border-gray-600 flex flex-col justify-between">
              <div>
                <h4 class="font-semibold text-lg mb-1" x-text="incentive.name"></h4>
                <p class="text-sm text-gray-600 dark:text-gray-300">Reward: <span x-text="incentive.reward"></span></p>
                <p class="text-xs mt-1">Progress: <span x-text="Math.round(incentive.progress * 10) + '/10'"></span></p>
                <div class="mt-2 w-full bg-gray-200 dark:bg-gray-600 rounded-full h-3">
                  <div class="bg-green-500 h-3 rounded-full transition-all duration-500" :style="'width: ' + (incentive.progress * 100) + '%'"></div>
                </div>
              </div>
              <button @click="incentive.accepted = !incentive.accepted" 
                      :class="incentive.accepted ? 'bg-green-500' : 'bg-indigo-500'"
                      class="mt-4 text-white px-4 py-2 rounded transition">
                <span x-text="incentive.accepted ? '✔️ Accepted' : 'Opt-in' "></span>
              </button>
            </div>
          </template>
        </div>
      </div>

      <div class="card">
        <h3 class="mb-4 text-xl font-semibold">Seus Badges</h3>
        <div class="flex gap-6 flex-wrap">
          <template x-for="badge in badges" :key="badge.id">
            <div class="flex flex-col items-center bg-white dark:bg-gray-700 p-4 rounded-lg shadow border border-gray-200 dark:border-gray-600 w-48" :title="badge.description">
              <img :src="badge.img" alt="badge" class="w-20 h-20 rounded-full object-cover mb-2" />
              <span class="text-sm font-medium" x-text="badge.name"></span>
            </div>
          </template>
        </div>
      </div>

      <div class="card">
        <h3 class="mb-4 text-xl font-semibold">Commissions per Product Type</h3>
        <canvas id="dealerChart" width="400" height="200"></canvas>
      </div>
    </section>
  </div>
 <script>
  document.addEventListener('alpine:init', () => {
    Alpine.data('dealerDashboard', () => ({
      tab: 'dealer',
      theme: 'light',
      showModal: null,
      badges: [
        { id: 'badge1', name: 'Partner', description: 'Participated actively during the month.', img: 'https://cdn-icons-png.flaticon.com/512/1322/1322195.png'},
        { id: 'badge2', name: 'Top Seller', description: 'Achieved high sales performance.', img: 'https://cdn-icons-png.flaticon.com/512/5996/5996680.png' },
        { id: 'badge3', name: 'Active Dealer', description: 'Completed all sales incentives.', img: 'https://cdn-icons-png.flaticon.com/512/3135/3135789.png' }
      ],
      incentives: [
        { id: 1, name: 'Sell 10 iPhones', reward: '$500', accepted: false, progress: 0.1 },
        { id: 2, name: 'Sell 10 Postpaid Plans', reward: '$10 per plan', accepted: false, progress: 0.3 },
        { id: 3, name: 'Sell 10 x 15GB plans', reward: 'spin the wheel', accepted: false, progress: 0.2 }
      ],
      newIncentive: { name: '', reward: '' },
      newBadge: { name: '', img: '' },
      dailyCommissionList: [
        'Sale#001 ($500) --> Commissions ($25.00)',
        'Sale#002 ($300) --> Commissions ($15.00)',
        'Sale#003 ($300) --> Commissions ($30.00)',
        'Sale#004 ($125) --> Commissions ($12.50)',
        'Sale#005 ($185) --> Commissions ($18.75)'
      ],
      awardList: [
        '$500.00 - iPhone Challenge Completed',
        '$150.00 - Postpaid Plans Challenge Completed',
        '$110.00 as Top Seller Bonus',
		'$100.00 as Partner Badge'
      ],
      topDealers: [
        { name: 'Dealer A', sales: 1410 },
        { name: 'Dealer B', sales: 450 },
        { name: 'Dealer C', sales: 400 },
        { name: 'Dealer D', sales: 350 },
        { name: 'Dealer E', sales: 300 },
        { name: 'Dealer F', sales: 250 },
        { name: 'Dealer G', sales: 200 },
        { name: 'Dealer H', sales: 130 },
        { name: 'Dealer I', sales: 100 },
        { name: 'Dealer J', sales: 50 }
      ],
      get dailyCommissions() {
        return this.incentives.reduce((sum, inc) => sum + (inc.progress * 100), +36.25);
      },
      get totalAwards() {
        return this.incentives
          .filter(i => i.progress === 1 && i.accepted)
          .reduce((total, i) => {
            const num = parseFloat(i.reward.replace(/[^0-9.-]+/g, ""));
            return total + (isNaN(num) ? 0 : num);
          }, 860);
      },
      get userRank() {
        return '1';
      },
      get acceptedIncentivesCount() {
        return this.incentives.filter(i => i.accepted).length;
      },
              init() {
          this.$nextTick(() => {
            const ctx = document.getElementById('dealerChart').getContext('2d');
            new Chart(ctx, {
              type: 'bar',
              data: {
                labels: ['Devices', 'Activations', 'Packages', 'Services'],
                datasets: [
                  {
                    label: 'sales',
                    data: [300, 150, 200, 100],
                    backgroundColor: 'rgba(54, 162, 235, 0.5)'
                  },
                  {
                    label: 'commissions',
                    data: [45, 20, 30, 10],
                    backgroundColor: 'rgba(255, 206, 86, 0.5)'
                  }
                ]
              },
              options: {
                responsive: true,
                plugins: {
                  legend: {
                    position: 'top'
                  },
                  title: {
                    display: true,
                    text: 'Sales x Commissions per Product Type'
                  }
                }
              }
            });
          });
        },
    }));
  });
 </script>
</body>
</html>
