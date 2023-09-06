# COLLECTION INVESTMENTS TRACKER


## HOW TO USE THE BLOCKSPAN API TO FIND COLLECTION SALES DATA

Blockspan is a leading provider of NFT API services, enabling developers to easily interact with the world of non-fungible tokens (NFTs). NFTs represent ownership of a unique item or piece of content on the blockchain. A collection investments tracking tool will provide users with an overview of recent sales data and how it has changed over the timeframe, including total sales, unique tokens sold, and average price. 


## REQUIREMENTS:
- Node.js and npm installed on your system.
- Basic knowledge of React.js
- Blockspan API key


## STEP 1: SET UP YOU REACT APPLICATION

First, you'll need to set up your React application. If you already have a React application set up, you can skip this step.

`npx create-react-app collection-investments-tracker` 
`cd collection-investments-tracker`

This will create a new React application named `collection-investments-tracker` and navigate into the new directory.


## STEP 2: INSTALL AXIOS

We'll be using Axios to send HTTP requests to the Blockspan API. Install it with the following command:

`npm install axios`


## STEP 3: CREATE YOUR REACT COMPONENT

Next, you'll need to create a React component that uses the Blockspan API to fetch portfolio data. Create a new file in the `src` directory called `investmentsTracker.js` and include the following code:

```
import React, { useState } from 'react';
import axios from 'axios';
import './App.css';

const InvestmentsTracker = () => {
  const [contractAddress, setcontractAddress] = useState('');
  const [blockchain, setBlockchain] = useState('eth-main');
  const [timeframe, setTimeframe] = useState('1_DAY');
  const [timeframeNumber, setTimeframeNumber] = useState(1);
  const [binsize, setBinsize] = useState('1_HOUR');
  const [todaysData, setTodaysData] = useState(null);
  const [previousData, setPreviousData] = useState(null);
  const [error, setError] = useState(false);
  const [loading, setLoading] = useState(false);
  const [hasClicked, setHasClicked] = useState(false);

  const bins = {
    '1_DAY': '1_HOUR',
    '7_DAYS': '1_DAY',
    '30_DAYS': '1_DAY',
  };

  const metrics = {
    'Total Sales': 'total_sales',
    'Unique Tokens': 'total_unique_tokens',
    'Average Price (USD)': 'avg_price_usd',
    'Max Price (USD)': 'max_price_usd',
    'Total Sales Volume (USD)': 'total_sales_volume_usd',
  };

  const retrieveData = async () => {
    if (!contractAddress.trim()) {
      setError('Contract address is required.');
      setTodaysData(null)
      setPreviousData(null)
      return;
    }

    setTodaysData(null);
    setPreviousData(null);
    setHasClicked(true);
    setLoading(true);

    const currentUTCDate = new Date();
    const currentUTCDateString = currentUTCDate.toISOString();

    if (timeframe === '1_DAY') {
      setTimeframeNumber(1);
    } else if (timeframe === '7_DAYS') {
      setTimeframeNumber(7);
    } else if (timeframe === '30_DAYS') {
      setTimeframeNumber(30);
    }

    const oldUTCDate = new Date(
      currentUTCDate.getTime() - timeframeNumber * 24 * 60 * 60 * 1000
    );
    const oldUTCDateString = oldUTCDate.toISOString();

    setBinsize(bins[timeframe]);
    const params = `contract_address=${contractAddress}&chain=${blockchain}&timeframe=${timeframe}&bin_size=${binsize}`;

    const url1 = `https://api.blockspan.com/v1/nfts/nfthistory?timestamp_end=${currentUTCDateString}&${params}`;
    const url2 = `https://api.blockspan.com/v1/nfts/nfthistory?timestamp_end=${oldUTCDateString}&${params}`;
    const headers = {
      accept: 'application/json',
      'X-API-KEY': 'YOUR_BLOCKSPAN_API_KEY',
    };

    try {
      const response1 = await axios.get(url1, { headers });
      const response2 = await axios.get(url2, { headers });
      setTodaysData(response1.data);
      console.log('Today:', todaysData);
      setPreviousData(response2.data);
      console.log('Previous:', previousData);
      setError(null);
      setLoading(false);
    } catch (error) {
      console.error(error);
      error.response.status === 401
        ? setError('Invalid blockspan API key!')
        : setError('Error: verify chain and contract address are valid');
      setTodaysData(null);
      setPreviousData(null);
      setLoading(false);
    }
  };

  function formatData(data) {
    if (typeof data === 'string' && !isNaN(Number(data))) {
      return Number(data).toFixed(2); // Round to two decimal places for floats
    } else {
      return data; // Leave it as is for non-floats
    }
  }

  function calculatePercentageChange(previousValue, currentValue) {
    if (
      typeof previousValue === 'number' &&
      typeof currentValue === 'number' &&
      !isNaN(previousValue) &&
      !isNaN(currentValue) &&
      previousValue !== 0
    ) {
      return ((currentValue / previousValue - 1) * 100).toFixed(2); // Calculate and round to two decimal places
    } else {
      return '';
    }
  }

  return (
    <div>
      <h1 className="title">Collection Investments Tracker</h1>
      <p className="message">
        Select a blockchain and timeframe, then input a contract address to see
        how sales data has changed.
      </p>
      <div className="inputContainer">
        <select
          name="blockchain"
          value={blockchain}
          onChange={(e) => setBlockchain(e.target.value)}
        >
          <option value="eth-main">eth-main</option>
          <option value="arbitrum-main">arbitrum-main</option>
          <option value="optimism-main">optimism-main</option>
          <option value="poly-main">poly-main</option>
          <option value="bsc-main">bsc-main</option>
          <option value="eth-goerli">eth-goerli</option>
        </select>
        <select
          name="timeframe"
          value={timeframe}
          onChange={(e) => {
            setTimeframe(e.target.value);
            setBinsize(bins[e.target.value]);
          }}
        >
          <option value="1_DAY">1 Day</option>
          <option value="7_DAYS">7 Days</option>
          <option value="30_DAYS">30 Days</option>
        </select>
        <input
          type="text"
          placeholder="Contract Address"
          value={contractAddress}
          onChange={(e) => setcontractAddress(e.target.value)}
        />
        <button onClick={retrieveData}>Retrieve Data</button>
      </div>
      {loading ? (
        <p className="message">Loading...</p>
      ) : (
        <>
          {error && <p className="errorMessage">{error}</p>}
          {hasClicked && (
            <>
              {todaysData?.total_sales === 0 && previousData?.total_sales === 0 ? (
                <p className="errorMessage">
                  No sales data found. Verify chain, address, and timeframe.
                </p>
              ) : (
                <div>
                  {todaysData !== null && previousData !== null && (
                    <>
                      <p className="message">
                        The current data represents sales data from today back to{' '}
                        {timeframeNumber}{' '}
                        {timeframeNumber === 1 ? 'day' : 'days'} ago.
                      </p>
                      <p className="message">
                        The data from {timeframeNumber}{' '}
                        {timeframeNumber === 1 ? 'day' : 'days'} ago is over the
                        same timeframe but starting {timeframeNumber} days ago.
                      </p>
                      <table className="tableContainer">
                        <thead>
                          <tr style={{ backgroundColor: '#f2f2f2' }}>
                            <th>Sales Data</th>
                            <th>
                              {timeframeNumber} day
                              {timeframeNumber === 1 ? '' : 's'} ago
                            </th>
                            <th>Today</th>
                            <th>Change</th>
                          </tr>
                        </thead>
                        <tbody>
                          {Object.keys(metrics).map((key) => (
                            <tr
                              style={{ backgroundColor: '#f2f2f2' }}
                              key={key}
                            >
                              <td>{key}</td>
                              <td>
                                {formatData(previousData[metrics[key]])}
                              </td>
                              <td>{formatData(todaysData[metrics[key]])}</td>
                              <td
                                style={{
                                  color:
                                    calculatePercentageChange(
                                      Number(previousData[metrics[key]]),
                                      Number(todaysData[metrics[key]])
                                    ) < 0
                                      ? 'red'
                                      : 'green',
                                }}
                              >
                                {calculatePercentageChange(
                                  Number(previousData[metrics[key]]),
                                  Number(todaysData[metrics[key]])
                                ) > 0
                                  ? `+${calculatePercentageChange(
                                      Number(previousData[metrics[key]]),
                                      Number(todaysData[metrics[key]])
                                    )}%`
                                  : `${calculatePercentageChange(
                                      Number(previousData[metrics[key]]),
                                      Number(todaysData[metrics[key]])
                                    )}%`}
                              </td>
                            </tr>
                          ))}
                        </tbody>
                      </table>
                    </>
                  )}
                </div>
              )}
            </>
          )}
        </>
      )}
    </div>
  );
};

export default InvestmentsTracker;

```

Remember to replace `YOUR_BLOCKSPAN_API_KEY` with your actual Blockspan API key.


## STEP 4: UPDATING THE STYLES WITHIN CSS FILE

To enhance the user interface in the browser, replace all code in the App.css file with the following:

```
.App {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  min-height: 100vh;
  overflow-y: auto;
}

.title {
  margin-top: 20px;
  margin-bottom: 0;
  text-align: center;
}

.errorMessage {
  text-align: center;
  color: red;
  font-weight: bold;
}

.successMessage {
  text-align: center;
  color: green;
  font-weight: bold;
}

.message {
  text-align: center;
}

.image {
  display: flex;
  justify-content: center;
  align-items: center;
}

.inputContainer {
  display: flex;
  justify-content: center;
  gap: 10px;
  margin-bottom: 20px;
}

.inputContainer input {
  padding: 10px;
  font-size: 1em;
  width: 200px;
}

.inputContainer button {
  padding: 10px;
  font-size: 1em;
  background-color: #007BFF;
  color: white;
  border: none;
  cursor: pointer;
}

.inputContainer button:hover {
  background-color: #0056b3;
}

.imageContainer {
  display: flex;
  justify-content: center;
  width: 100%; 
}

.imageContainer img {
  width: 100%; 
  max-width: 500px;
  height: auto; 
}
.nftData {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-top: 20px;
}

.nftData .image {
  display: flex;
  justify-content: center;
  align-items: center;
}

.nftData h2 {
  margin: 10px 0;
}

.nftData p {
  font-size: 1.2em;
  font-weight: bold;
}

td {
  padding: 10px;
  text-align: center;
}

th {
  padding: 10px;
  text-align: center;
}

.tableContainer {
  width: 100%;
  border-collapse: separate;
  border-spacing: 4px; 
}

```


## STEP 5: INTEGRATING COMPONENTS IN THE APP

Finally, let's use the `InvestmentsTracker` component in our main `App` component.

Open App.js and modify it as follows:

```
import React from 'react';
import './App.css';
import InvestmentsTracker from './investmentsTracker';

function App() {
  return (
    <div className="App">
      <InvestmentsTracker/>
    </div>
  );
}

export default App;
```

Now, start the app with the following command:

`npm start`

You should now see the following:

- A drop down menu to select a blockchain
- A drop down menu to select timeframe
- A text box for contract address
- A retrieve data button

Input the chain and contract address for the collection you want data for, and click the retrieve data button. You should then see a grey table with the collections recent sales data trends if applicable. 

## CONCLUSION

Congratulations! You've just built a simple yet powerful collection investments tracker using the Blockspan API and React.js. As you've seen, the Blockspan API is intuitive to use and provides detailed and accurate information, making it a perfect choice for this kind of application. This tutorial is just a starting point - there are many ways you can expand and improve your tool. For example, you could add more error checking, improve the UI, or display additional data.

As the world of NFTs continues to grow and evolve, tools like this will become increasingly important. Whether you're an NFT enthusiast, a developer, or a startup, understanding NFTs is a valuable skill. We hope you found this tutorial helpful.
