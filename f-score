import pandas as pd
import requests
import timeit


start = timeit.default_timer()


def statement(ticker, statement, period):
    if statement == 'IS':
        stmt = 'income-statement'
    elif statement == 'BS':
        stmt = 'balance-sheet-statement'
    else:
        stmt = 'cash-flow-statement'
    if period == 'Q':
        prd = 'quarter'
    else:
        prd = 'annual'

    fs = requests.get(f'https://financialmodelingprep.com/api/v3/financials/'+stmt+'/'+ticker+'?period='+prd)
    fs = fs.json()

    fs = fs['financials']
    fs = pd.DataFrame.from_dict(fs)
    fs = fs.T
    fs.columns = fs.iloc[0]

    fs = fs.iloc[1:]
    fs = fs.iloc[:, :4]

    return fs


def piotroski_score(ticker):
    df_is = pd.DataFrame(statement(ticker, 'IS', 'A'))
    df_bs = pd.DataFrame(statement(ticker, 'BS', 'A'))
    df_cf = pd.DataFrame(statement(ticker, 'CF', 'A'))

    def net_income():
        return float(df_is.loc['Net Income'][0])

    def roa():
        p, q = float(df_bs.loc['Total assets'][0]), float(df_bs.loc['Total assets'][1])
        avg_assets = (p + q) / 2
        return net_income() / avg_assets

    def ocf():
        return float(df_cf.loc['Operating Cash Flow'][0])

    def ltdebt_chg():
        p, q = float(df_bs.loc['Long-term debt'][0]), float(df_bs.loc['Long-term debt'][1])
        return q - p

    def current_ratio_chg():
        p, q = float(df_bs.loc['Total current assets'][0]), float(df_bs.loc['Total current assets'][1])
        r, s = float(df_bs.loc['Total current liabilities'][0]), float(df_bs.loc['Total current liabilities'][1])
        current_ratio1, current_ratio2 = p / r, q / s
        return current_ratio1 - current_ratio2

    def new_shares():
        p, q = float(df_bs.loc['Retained earnings (deficit)'][0]), float(df_bs.loc['Retained earnings (deficit)'][1])
        r, s = float(df_bs.loc['Total shareholders equity'][0]), float(df_bs.loc['Total shareholders equity'][1])
        return (s - q) - (r - p)

    def gross_margin_chg():
        p, q = float(df_is.loc['Gross Margin'][0]), float(df_is.loc['Gross Margin'][1])
        return p - q

    def asset_turnover_ratio_chg():
        p, q, s = float(df_bs.loc['Total assets'][0]), float(df_bs.loc['Total assets'][1]), \
            float(df_bs.loc['Total assets'][2])
        avg_assets1 = (p + q) / 2
        avg_assets2 = (q + s) / 2
        atr1 = float(df_is.loc['Revenue'][0]) / avg_assets1
        atr2 = float(df_is.loc['Revenue'][1]) / avg_assets2
        return atr1 - atr2

    pScore = 0
    if net_income() > 0:
        pScore += 1
    if roa() > 0:
        pScore += 1
    if ocf() > 0:
        pScore += 1
    if ocf() > net_income():
        pScore += 1
    if ltdebt_chg() > 0:
        pScore += 1
    if current_ratio_chg() > 0:
        pScore += 1
    if new_shares() > 0:
        pScore += 1
    if gross_margin_chg() > 0:
        pScore += 1
    if asset_turnover_ratio_chg() > 0:
        pScore += 1
    return pScore


tickers = ['AAPL', 'GOOG', 'TSLA', 'F', 'MSFT', 'MS']
data2 = {}
for ticker in tickers:
    data2[ticker] = piotroski_score(ticker)
    data = pd.DataFrame(data2, index=[0])


def header(msg):
    print('-' * 50)
    print('[' + msg + ']')


print('F Score report for: ')
print(tickers)
print(data)

stop = timeit.default_timer()
print('Time: ', stop - start)
