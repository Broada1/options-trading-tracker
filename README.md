[options-trading-tracker (2).tsx](https://github.com/user-attachments/files/23200518/options-trading-tracker.2.tsx)
import React, { useState, useEffect } from 'react';
import { Plus, Trash2, Save, TrendingUp, DollarSign, Target, PieChart } from 'lucide-react';

const OptionsTradingTracker = () => {
  const [activeTab, setActiveTab] = useState('dashboard');
  const [loading, setLoading] = useState(true);
  
  const [budget, setBudget] = useState({
    monthlyIncome: 5200,
    monthlyExpenses: 3784,
    investmentAllocation: 1416
  });

  const [positions, setPositions] = useState([]);
  
  const [strikeCalc, setStrikeCalc] = useState({
    ticker: 'AMD',
    currentPrice: 245.66,
    correction: 10,
    threeMonthHigh: 186.65,
    sixMonthHigh: 186.65,
    twelveMonthHigh: 186.65,
    twentyFourMonthHigh: 227.30,
    thirtySixMonthHigh: 227.30
  });

  useEffect(() => {
    loadData();
  }, []);

  const loadData = async () => {
    try {
      const budgetResult = await window.storage.get('budget').catch(() => null);
      const positionsResult = await window.storage.get('positions').catch(() => null);
      const strikeResult = await window.storage.get('strikeCalc').catch(() => null);

      if (budgetResult && budgetResult.value) {
        setBudget(JSON.parse(budgetResult.value));
      }
      if (positionsResult && positionsResult.value) {
        setPositions(JSON.parse(positionsResult.value));
      }
      if (strikeResult && strikeResult.value) {
        setStrikeCalc(JSON.parse(strikeResult.value));
      }
    } catch (error) {
      console.log('No saved data found, using defaults');
    } finally {
      setLoading(false);
    }
  };

  const saveData = async () => {
    try {
      await window.storage.set('budget', JSON.stringify(budget));
      await window.storage.set('positions', JSON.stringify(positions));
      await window.storage.set('strikeCalc', JSON.stringify(strikeCalc));
      alert('Data saved successfully!');
    } catch (error) {
      console.error('Save error:', error);
      alert('Error saving data. Please try again.');
    }
  };

  const calculateStrikes = (timeframe) => {
    const cp = strikeCalc.currentPrice;
    const corr = strikeCalc.correction;
    const correctionFactor = 1 - (corr / 100);
    
    return {
      itm: Math.round(cp * 1.08),
      conservative: Math.round(cp * 1.18),
      mid: Math.round(cp * 1.28),
      aggressive: Math.round(cp * 1.20),
      corrected: Math.round(cp * correctionFactor * 0.9)
    };
  };

  const addPosition = () => {
    const newPosition = {
      id: Date.now(),
      ticker: '',
      type: 'call',
      strike: 0,
      expiration: '',
      contracts: 1,
      premium: 0,
      entryDate: new Date().toISOString().split('T')[0],
      status: 'open',
      exitDate: '',
      exitPrice: 0,
      notes: ''
    };
    setPositions([...positions, newPosition]);
  };

  const updatePosition = (id, field, value) => {
    setPositions(positions.map(pos => {
      if (pos.id === id) {
        const updated = {...pos};
        updated[field] = value;
        return updated;
      }
      return pos;
    }));
  };

  const deletePosition = (id) => {
    setPositions(positions.filter(pos => pos.id !== id));
  };

  const closePosition = (id, exitPrice) => {
    setPositions(positions.map(pos => {
      if (pos.id === id) {
        return {
          ...pos,
          status: 'closed',
          exitDate: new Date().toISOString().split('T')[0],
          exitPrice: parseFloat(exitPrice) || 0
        };
      }
      return pos;
    }));
  };

  const calculatePL = (position) => {
    if (position.status === 'closed') {
      const profit = (position.exitPrice - position.premium) * position.contracts * 100;
      return profit;
    }
    return 0;
  };

  const calculatePortfolioMetrics = () => {
    const openPositions = positions.filter(p => p.status === 'open');
    const closedPositions = positions.filter(p => p.status === 'closed');
    
    const totalInvested = openPositions.reduce((sum, p) => {
      return sum + (p.premium * p.contracts * 100);
    }, 0);
    
    const realizedPL = closedPositions.reduce((sum, p) => {
      return sum + calculatePL(p);
    }, 0);
    
    const totalCost = positions.reduce((sum, p) => {
      return sum + (p.premium * p.contracts * 100);
    }, 0);

    const roi = totalCost > 0 ? ((realizedPL / totalCost) * 100).toFixed(2) : 0;

    return {
      openPositions: openPositions.length,
      closedPositions: closedPositions.length,
      totalInvested: totalInvested,
      realizedPL: realizedPL,
      totalCost: totalCost,
      roi: roi
    };
  };

  const metrics = calculatePortfolioMetrics();

  if (loading) {
    return (
      <div className="flex items-center justify-center h-screen bg-gray-900">
        <div className="text-white text-xl">Loading your trading data...</div>
      </div>
    );
  }

  return (
    <div className="min-h-screen bg-gray-900 text-white p-4">
      <div className="max-w-7xl mx-auto">
        <div className="flex justify-between items-center mb-6">
          <h1 className="text-3xl font-bold">Options Trading Plan Tracker</h1>
          <button 
            onClick={saveData}
            className="flex items-center gap-2 bg-green-600 hover:bg-green-700 px-4 py-2 rounded-lg transition"
          >
            <Save size={20} />
            Save All Data
          </button>
        </div>

        <div className="flex gap-2 mb-6 overflow-x-auto">
          {['dashboard', 'positions', 'strikes', 'budget', 'performance'].map(tab => (
            <button
              key={tab}
              onClick={() => setActiveTab(tab)}
              className={`px-4 py-2 rounded-lg capitalize whitespace-nowrap transition ${
                activeTab === tab 
                  ? 'bg-blue-600 text-white' 
                  : 'bg-gray-800 text-gray-400 hover:bg-gray-700'
              }`}
            >
              {tab}
            </button>
          ))}
        </div>

        {activeTab === 'dashboard' && (
          <div className="space-y-6">
            <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4">
              <div className="bg-gray-800 p-6 rounded-lg">
                <div className="flex items-center justify-between mb-2">
                  <span className="text-gray-400">Open Positions</span>
                  <TrendingUp className="text-blue-500" size={24} />
                </div>
                <div className="text-3xl font-bold">{metrics.openPositions}</div>
              </div>
              
              <div className="bg-gray-800 p-6 rounded-lg">
                <div className="flex items-center justify-between mb-2">
                  <span className="text-gray-400">Total Invested</span>
                  <DollarSign className="text-green-500" size={24} />
                </div>
                <div className="text-3xl font-bold">${metrics.totalInvested.toFixed(2)}</div>
              </div>
              
              <div className="bg-gray-800 p-6 rounded-lg">
                <div className="flex items-center justify-between mb-2">
                  <span className="text-gray-400">Realized P&L</span>
                  <Target className="text-purple-500" size={24} />
                </div>
                <div className={`text-3xl font-bold ${metrics.realizedPL >= 0 ? 'text-green-500' : 'text-red-500'}`}>
                  ${metrics.realizedPL.toFixed(2)}
                </div>
              </div>
              
              <div className="bg-gray-800 p-6 rounded-lg">
                <div className="flex items-center justify-between mb-2">
                  <span className="text-gray-400">ROI</span>
                  <PieChart className="text-yellow-500" size={24} />
                </div>
                <div className={`text-3xl font-bold ${metrics.roi >= 0 ? 'text-green-500' : 'text-red-500'}`}>
                  {metrics.roi}%
                </div>
              </div>
            </div>

            <div className="bg-gray-800 p-6 rounded-lg">
              <h2 className="text-xl font-bold mb-4">Budget Overview (70/30 Goal)</h2>
              <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
                <div>
                  <div className="text-gray-400 text-sm">Monthly Income</div>
                  <div className="text-2xl font-bold">${budget.monthlyIncome.toLocaleString()}</div>
                </div>
                <div>
                  <div className="text-gray-400 text-sm">Monthly Expenses</div>
                  <div className="text-2xl font-bold">${budget.monthlyExpenses.toLocaleString()}</div>
                </div>
                <div>
                  <div className="text-gray-400 text-sm">Available to Invest</div>
                  <div className="text-2xl font-bold text-green-500">${budget.investmentAllocation.toLocaleString()}</div>
                </div>
              </div>
              <div className="mt-4">
                <div className="flex justify-between text-sm mb-2">
                  <span>Current: {((budget.investmentAllocation / budget.monthlyIncome) * 100).toFixed(1)}% invested</span>
                  <span>Goal: 70% invested</span>
                </div>
                <div className="w-full bg-gray-700 rounded-full h-4">
                  <div 
                    className="bg-blue-600 h-4 rounded-full transition-all"
                    style={{ width: `${Math.min((budget.investmentAllocation / budget.monthlyIncome) * 100, 100)}%` }}
                  ></div>
                </div>
              </div>
            </div>

            <div className="bg-gray-800 p-6 rounded-lg">
              <h2 className="text-xl font-bold mb-4">Recent Positions</h2>
              {positions.length === 0 ? (
                <div className="text-gray-400 text-center py-8">No positions yet. Add your first position in the Positions tab.</div>
              ) : (
                <div className="overflow-x-auto">
                  <table className="w-full">
                    <thead>
                      <tr className="text-left text-gray-400 border-b border-gray-700">
                        <th className="pb-2">Ticker</th>
                        <th className="pb-2">Type</th>
                        <th className="pb-2">Strike</th>
                        <th className="pb-2">Expiration</th>
                        <th className="pb-2">Status</th>
                        <th className="pb-2">P&L</th>
                      </tr>
                    </thead>
                    <tbody>
                      {positions.slice(0, 5).map(pos => (
                        <tr key={pos.id} className="border-b border-gray-700">
                          <td className="py-2 font-bold">{pos.ticker || 'N/A'}</td>
                          <td className="py-2 capitalize">{pos.type}</td>
                          <td className="py-2">${pos.strike}</td>
                          <td className="py-2">{pos.expiration || 'N/A'}</td>
                          <td className="py-2">
                            <span className={`px-2 py-1 rounded text-xs ${
                              pos.status === 'open' ? 'bg-green-900 text-green-300' : 'bg-gray-700 text-gray-300'
                            }`}>
                              {pos.status}
                            </span>
                          </td>
                          <td className={`py-2 font-bold ${calculatePL(pos) >= 0 ? 'text-green-500' : 'text-red-500'}`}>
                            ${calculatePL(pos).toFixed(2)}
                          </td>
                        </tr>
                      ))}
                    </tbody>
                  </table>
                </div>
              )}
            </div>
          </div>
        )}

        {activeTab === 'positions' && (
          <div className="space-y-4">
            <div className="flex justify-between items-center">
              <h2 className="text-2xl font-bold">Manage Positions</h2>
              <button 
                onClick={addPosition}
                className="flex items-center gap-2 bg-blue-600 hover:bg-blue-700 px-4 py-2 rounded-lg transition"
              >
                <Plus size={20} />
                Add Position
              </button>
            </div>

            {positions.length === 0 ? (
              <div className="bg-gray-800 p-12 rounded-lg text-center">
                <p className="text-gray-400 mb-4">No positions added yet</p>
                <button 
                  onClick={addPosition}
                  className="bg-blue-600 hover:bg-blue-700 px-6 py-3 rounded-lg transition"
                >
                  Add Your First Position
                </button>
              </div>
            ) : (
              <div className="space-y-4">
                {positions.map(pos => (
                  <div key={pos.id} className="bg-gray-800 p-6 rounded-lg">
                    <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4 mb-4">
                      <div>
                        <label className="block text-sm text-gray-400 mb-1">Ticker</label>
                        <input
                          type="text"
                          value={pos.ticker}
                          onChange={(e) => updatePosition(pos.id, 'ticker', e.target.value.toUpperCase())}
                          className="w-full bg-gray-700 rounded px-3 py-2"
                          placeholder="AMD"
                        />
                      </div>
                      
                      <div>
                        <label className="block text-sm text-gray-400 mb-1">Type</label>
                        <select
                          value={pos.type}
                          onChange={(e) => updatePosition(pos.id, 'type', e.target.value)}
                          className="w-full bg-gray-700 rounded px-3 py-2"
                        >
                          <option value="call">Call</option>
                          <option value="put">Put</option>
                        </select>
                      </div>
                      
                      <div>
                        <label className="block text-sm text-gray-400 mb-1">Strike Price</label>
                        <input
                          type="number"
                          value={pos.strike}
                          onChange={(e) => updatePosition(pos.id, 'strike', parseFloat(e.target.value) || 0)}
                          className="w-full bg-gray-700 rounded px-3 py-2"
                        />
                      </div>
                      
                      <div>
                        <label className="block text-sm text-gray-400 mb-1">Expiration</label>
                        <input
                          type="date"
                          value={pos.expiration}
                          onChange={(e) => updatePosition(pos.id, 'expiration', e.target.value)}
                          className="w-full bg-gray-700 rounded px-3 py-2"
                        />
                      </div>
                      
                      <div>
                        <label className="block text-sm text-gray-400 mb-1">Contracts</label>
                        <input
                          type="number"
                          value={pos.contracts}
                          onChange={(e) => updatePosition(pos.id, 'contracts', parseInt(e.target.value) || 1)}
                          className="w-full bg-gray-700 rounded px-3 py-2"
                          min="1"
                        />
                      </div>
                      
                      <div>
                        <label className="block text-sm text-gray-400 mb-1">Premium Paid</label>
                        <input
                          type="number"
                          step="0.01"
                          value={pos.premium}
                          onChange={(e) => updatePosition(pos.id, 'premium', parseFloat(e.target.value) || 0)}
                          className="w-full bg-gray-700 rounded px-3 py-2"
                        />
                      </div>
                      
                      <div>
                        <label className="block text-sm text-gray-400 mb-1">Entry Date</label>
                        <input
                          type="date"
                          value={pos.entryDate}
                          onChange={(e) => updatePosition(pos.id, 'entryDate', e.target.value)}
                          className="w-full bg-gray-700 rounded px-3 py-2"
                        />
                      </div>
                      
                      <div>
                        <label className="block text-sm text-gray-400 mb-1">Status</label>
                        <div className="flex gap-2">
                          <span className={`px-3 py-2 rounded ${
                            pos.status === 'open' ? 'bg-green-900 text-green-300' : 'bg-gray-700 text-gray-300'
                          }`}>
                            {pos.status}
                          </span>
                          {pos.status === 'open' && (
                            <button
                              onClick={() => {
                                const exitPrice = prompt('Enter exit price:');
                                if (exitPrice) closePosition(pos.id, exitPrice);
                              }}
                              className="bg-yellow-600 hover:bg-yellow-700 px-3 py-2 rounded text-sm transition"
                            >
                              Close
                            </button>
                          )}
                        </div>
                      </div>
                    </div>
                    
                    {pos.status === 'closed' && (
                      <div className="grid grid-cols-1 md:grid-cols-2 gap-4 mb-4">
                        <div>
                          <label className="block text-sm text-gray-400 mb-1">Exit Price</label>
                          <input
                            type="number"
                            step="0.01"
                            value={pos.exitPrice}
                            onChange={(e) => updatePosition(pos.id, 'exitPrice', parseFloat(e.target.value) || 0)}
                            className="w-full bg-gray-700 rounded px-3 py-2"
                          />
                        </div>
                        <div>
                          <label className="block text-sm text-gray-400 mb-1">Exit Date</label>
                          <input
                            type="date"
                            value={pos.exitDate}
                            onChange={(e) => updatePosition(pos.id, 'exitDate', e.target.value)}
                            className="w-full bg-gray-700 rounded px-3 py-2"
                          />
                        </div>
                      </div>
                    )}
                    
                    <div className="mb-4">
                      <label className="block text-sm text-gray-400 mb-1">Notes</label>
                      <textarea
                        value={pos.notes}
                        onChange={(e) => updatePosition(pos.id, 'notes', e.target.value)}
                        className="w-full bg-gray-700 rounded px-3 py-2"
                        rows="2"
                        placeholder="Trade rationale, strategy, etc."
                      />
                    </div>
                    
                    <div className="flex justify-between items-center pt-4 border-t border-gray-700">
                      <div>
                        <div className="text-sm text-gray-400">Total Cost</div>
                        <div className="text-xl font-bold">${(pos.premium * pos.contracts * 100).toFixed(2)}</div>
                      </div>
                      {pos.status === 'closed' && (
                        <div>
                          <div className="text-sm text-gray-400">P&L</div>
                          <div className={`text-xl font-bold ${calculatePL(pos) >= 0 ? 'text-green-500' : 'text-red-500'}`}>
                            ${calculatePL(pos).toFixed(2)}
                          </div>
                        </div>
                      )}
                      <button
                        onClick={() => deletePosition(pos.id)}
                        className="flex items-center gap-2 bg-red-600 hover:bg-red-700 px-4 py-2 rounded-lg transition"
                      >
                        <Trash2 size={16} />
                        Delete
                      </button>
                    </div>
                  </div>
                ))}
              </div>
            )}
          </div>
        )}

        {activeTab === 'strikes' && (
          <div className="space-y-6">
            <h2 className="text-2xl font-bold">Strike Price Calculator</h2>
            
            <div className="bg-gray-800 p-6 rounded-lg">
              <h3 className="text-xl font-bold mb-4">Input Data</h3>
              <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
                <div>
                  <label className="block text-sm text-gray-400 mb-1">Ticker</label>
                  <input
                    type="text"
                    value={strikeCalc.ticker}
                    onChange={(e) => setStrikeCalc({...strikeCalc, ticker: e.target.value.toUpperCase()})}
                    className="w-full bg-gray-700 rounded px-3 py-2"
                  />
                </div>
                <div>
                  <label className="block text-sm text-gray-400 mb-1">Current Price</label>
                  <input
                    type="number"
                    step="0.01"
                    value={strikeCalc.currentPrice}
                    onChange={(e) => setStrikeCalc({...strikeCalc, currentPrice: parseFloat(e.target.value) || 0})}
                    className="w-full bg-gray-700 rounded px-3 py-2"
                  />
                </div>
                <div>
                  <label className="block text-sm text-gray-400 mb-1">Correction %</label>
                  <input
                    type="number"
                    value={strikeCalc.correction}
                    onChange={(e) => setStrikeCalc({...strikeCalc, correction: parseFloat(e.target.value) || 0})}
                    className="w-full bg-gray-700 rounded px-3 py-2"
                  />
                </div>
              </div>
            </div>

            <div className="bg-gray-800 p-6 rounded-lg">
              <h3 className="text-xl font-bold mb-4">Calculated Strike Prices</h3>
              <div className="overflow-x-auto">
                <table className="w-full">
                  <thead>
                    <tr className="text-left border-b border-gray-700">
                      <th className="pb-3">Timeframe</th>
                      <th className="pb-3">In The Money</th>
                      <th className="pb-3">Conservative</th>
                      <th className="pb-3">Mid</th>
                      <th className="pb-3">Aggressive</th>
                      <th className="pb-3">With Correction</th>
                    </tr>
                  </thead>
                  <tbody>
                    {['3m', '6m', '12m', '24m', '36m'].map(timeframe => {
                      const strikes = calculateStrikes(timeframe);
                      return (
                        <tr key={timeframe} className="border-b border-gray-700">
                          <td className="py-3 font-bold">{timeframe.toUpperCase()}</td>
                          <td className="py-3">${strikes.itm}</td>
                          <td className="py-3 text-blue-400">${strikes.conservative}</td>
                          <td className="py-3 text-green-400">${strikes.mid}</td>
                          <td className="py-3 text-yellow-400">${strikes.aggressive}</td>
                          <td className="py-3 text-red-400">${strikes.corrected}</td>
                        </tr>
                      );
                    })}
                  </tbody>
                </table>
              </div>
            </div>
          </div>
        )}

        {activeTab === 'budget' && (
          <div className="space-y-6">
            <h2 className="text-2xl font-bold">Budget Management (70/30 Life)</h2>
            
            <div className="bg-gray-800 p-6 rounded-lg">
              <h3 className="text-xl font-bold mb-4">Monthly Overview</h3>
              <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
                <div>
                  <label className="block text-sm text-gray-400 mb-1">Monthly Income</label>
                  <input
                    type="number"
                    value={budget.monthlyIncome}
                    onChange={(e) => {
                      const income = parseFloat(e.target.value) || 0;
                      const allocation = income - budget.monthlyExpenses;
                      setBudget({...budget, monthlyIncome: income, investmentAllocation: allocation});
                    }}
                    className="w-full bg-gray-700 rounded px-3 py-2 text-xl font-bold"
                  />
                </div>
                <div>
                  <label className="block text-sm text-gray-400 mb-1">Monthly Expenses</label>
                  <input
                    type="number"
                    value={budget.monthlyExpenses}
                    onChange={(e) => {
                      const expenses = parseFloat(e.target.value) || 0;
                      const allocation = budget.monthlyIncome - expenses;
                      setBudget({...budget, monthlyExpenses: expenses, investmentAllocation: allocation});
                    }}
                    className="w-full bg-gray-700 rounded px-3 py-2 text-xl font-bold"
                  />
                </div>
                <div>
                  <label className="block text-sm text-gray-400 mb-1">Available to Invest</label>
                  <div className="w-full bg-gray-700 rounded px-3 py-2 text-xl font-bold text-green-500">
                    ${budget.investmentAllocation.toLocaleString()}
                  </div>
                </div>
              </div>
            </div>

            <div className="bg-gray-800 p-6 rounded-lg">
              <h3 className="text-xl font-bold mb-4">70/30 Progress</h3>
              <div className="w-full bg-gray-700 rounded-full h-6 mb-4">
                <div 
                  className="bg-blue-600 h-6 rounded-full flex items-center justify-center text-sm font-bold transition-all"
                  style={{ width: Math.min((budget.investmentAllocation / budget.monthlyIncome) * 100, 100) + '%' }}
                >
                  {((budget.investmentAllocation / budget.monthlyIncome) * 100).toFixed(1)}%
                </div>
              </div>
              <p className="text-gray-400">Goal: 70% invested | Annual capacity: ${(budget.investmentAllocation * 12).toLocaleString()}</p>
            </div>
          </div>
        )}

        {activeTab === 'performance' && (
          <div className="space-y-6">
            <h2 className="text-2xl font-bold">Performance Analytics</h2>

            <div className="grid grid-cols-1 md:grid-cols-4 gap-4">
              <div className="bg-gray-800 p-6 rounded-lg">
                <div className="text-gray-400 text-sm mb-1">Total Positions</div>
                <div className="text-3xl font-bold">{positions.length}</div>
              </div>

              <div className="bg-gray-800 p-6 rounded-lg">
                <div className="text-gray-400 text-sm mb-1">Capital Deployed</div>
                <div className="text-3xl font-bold">${metrics.totalCost.toFixed(2)}</div>
              </div>

              <div className="bg-gray-800 p-6 rounded-lg">
                <div className="text-gray-400 text-sm mb-1">Realized P&L</div>
                <div className={`text-3xl font-bold ${metrics.realizedPL >= 0 ? 'text-green-500' : 'text-red-500'}`}>
                  ${metrics.realizedPL.toFixed(2)}
                </div>
              </div>

              <div className="bg-gray-800 p-6 rounded-lg">
                <div className="text-gray-400 text-sm mb-1">ROI</div>
                <div className={`text-3xl font-bold ${metrics.roi >= 0 ? 'text-green-500' : 'text-red-500'}`}>
                  {metrics.roi}%
                </div>
              </div>
            </div>

            <div className="bg-gray-800 p-6 rounded-lg">
              <h3 className="text-xl font-bold mb-4">All Positions</h3>
              {positions.length === 0 ? (
                <div className="text-gray-400 text-center py-8">No positions to display</div>
              ) : (
                <div className="overflow-x-auto">
                  <table className="w-full">
                    <thead>
                      <tr className="text-left text-gray-400 border-b border-gray-700">
                        <th className="pb-3">Ticker</th>
                        <th className="pb-3">Type</th>
                        <th className="pb-3">Strike</th>
                        <th className="pb-3">Contracts</th>
                        <th className="pb-3">Premium</th>
                        <th className="pb-3">Cost</th>
                        <th className="pb-3">Status</th>
                        <th className="pb-3">P&L</th>
                        <th className="pb-3">ROI</th>
                      </tr>
                    </thead>
                    <tbody>
                      {positions.map(pos => {
                        const cost = pos.premium * pos.contracts * 100;
                        const pl = calculatePL(pos);
                        const roi = cost > 0 ? ((pl / cost) * 100).toFixed(2) : 0;
                        return (
                          <tr key={pos.id} className="border-b border-gray-700">
                            <td className="py-3 font-bold">{pos.ticker || 'N/A'}</td>
                            <td className="py-3 capitalize">{pos.type}</td>
                            <td className="py-3">${pos.strike}</td>
                            <td className="py-3">{pos.contracts}</td>
                            <td className="py-3">${pos.premium.toFixed(2)}</td>
                            <td className="py-3">${cost.toFixed(2)}</td>
                            <td className="py-3">
                              <span className={`px-2 py-1 rounded text-xs ${
                                pos.status === 'open' ? 'bg-green-900 text-green-300' : 'bg-gray-700 text-gray-300'
                              }`}>
                                {pos.status}
                              </span>
                            </td>
                            <td className={`py-3 font-bold ${pl >= 0 ? 'text-green-500' : 'text-red-500'}`}>
                              {pos.status === 'closed' ? `${pl.toFixed(2)}` : '-'}
                            </td>
                            <td className={`py-3 font-bold ${roi >= 0 ? 'text-green-500' : 'text-red-500'}`}>
                              {pos.status === 'closed' ? `${roi}%` : '-'}
                            </td>
                          </tr>
                        );
                      })}
                    </tbody>
                  </table>
                </div>
              )}
            </div>

            <div className="bg-gray-800 p-6 rounded-lg">
              <h3 className="text-xl font-bold mb-4">Win Rate Analysis</h3>
              <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
                {(() => {
                  const closedPos = positions.filter(p => p.status === 'closed');
                  const winners = closedPos.filter(p => calculatePL(p) > 0).length;
                  const losers = closedPos.filter(p => calculatePL(p) < 0).length;
                  const winRate = closedPos.length > 0 ? ((winners / closedPos.length) * 100).toFixed(1) : 0;
                  
                  return (
                    <>
                      <div className="text-center p-4 bg-gray-700 rounded-lg">
                        <div className="text-gray-400 text-sm mb-1">Winners</div>
                        <div className="text-3xl font-bold text-green-500">{winners}</div>
                        <div className="text-xs text-gray-500 mt-1">Profitable trades</div>
                      </div>
                      <div className="text-center p-4 bg-gray-700 rounded-lg">
                        <div className="text-gray-400 text-sm mb-1">Losers</div>
                        <div className="text-3xl font-bold text-red-500">{losers}</div>
                        <div className="text-xs text-gray-500 mt-1">Losing trades</div>
                      </div>
                      <div className="text-center p-4 bg-gray-700 rounded-lg">
                        <div className="text-gray-400 text-sm mb-1">Win Rate</div>
                        <div className="text-3xl font-bold text-blue-500">{winRate}%</div>
                        <div className="text-xs text-gray-500 mt-1">Success rate</div>
                      </div>
                    </>
                  );
                })()}
              </div>
            </div>

            <div className="bg-gray-800 p-6 rounded-lg">
              <h3 className="text-xl font-bold mb-4">Trade Notes</h3>
              {positions.filter(p => p.notes).length === 0 ? (
                <div className="text-gray-400 text-center py-8">No trade notes yet. Add notes to your positions.</div>
              ) : (
                <div className="space-y-4">
                  {positions.filter(p => p.notes).map(pos => (
                    <div key={pos.id} className="p-4 bg-gray-700 rounded-lg">
                      <div className="flex justify-between items-start mb-2">
                        <div>
                          <span className="font-bold text-lg">{pos.ticker}</span>
                          <span className="text-gray-400 ml-2">${pos.strike} {pos.type}</span>
                        </div>
                        <span className={`px-2 py-1 rounded text-xs ${
                          pos.status === 'open' ? 'bg-green-900 text-green-300' : 'bg-gray-600 text-gray-300'
                        }`}>
                          {pos.status}
                        </span>
                      </div>
                      <div className="text-gray-300 text-sm whitespace-pre-wrap">{pos.notes}</div>
                      {pos.status === 'closed' && (
                        <div className="mt-2 pt-2 border-t border-gray-600 flex justify-between text-sm">
                          <span className="text-gray-400">Result:</span>
                          <span className={`font-bold ${calculatePL(pos) >= 0 ? 'text-green-500' : 'text-red-500'}`}>
                            ${calculatePL(pos).toFixed(2)}
                          </span>
                        </div>
                      )}
                    </div>
                  ))}
                </div>
              )}
            </div>
          </div>
        )}
      </div>
    </div>
  );
};

export default OptionsTradingTracker;
