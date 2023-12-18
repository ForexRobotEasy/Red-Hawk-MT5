//+------------------------------------------------------------------+
//|                                                       Red Hawk MT5 |
//|                                         Developer: Forex Robot Easy Team |
//|                                                  Website: forexroboteasy.com |
//+------------------------------------------------------------------+

// Define input parameters
// These are the user-defined input parameters that can be adjusted to customize the trading strategy
input double StopLossPercentage = 1.5; // Stop loss percentage
input double TakeProfitPercentage = 3.0; // Take profit percentage
input double TrailingStopPercentage = 0.5; // Trailing stop percentage
input double MaxSpread = 2.0; // Maximum allowed spread in pips

// Define currency pairs
// These are the currency pairs that the robot will trade
string[] CurrencyPairs = {'EURUSD', 'GBPUSD', 'USDCHF', 'EURCHF', 'EURGBP', 'AUCAD', 'AUDJPY', 'EURAUD', 'USDCAD'};

//+------------------------------------------------------------------+
//| Expert initialization function                                   |
//+------------------------------------------------------------------+
// This function is called once when the expert advisor is first initialized
int OnInit()
{
    // Enable hedging
    Hedging(true);
    
    // Set optimization parameters for low spread and fast execution
    SetOptimizationParameter(OPTIMIZATION_CRITERION, 1);
    SetOptimizationParameter(OPTIMIZATION_GENE, 0);
    
    // Add filters to avoid trading during high spread or hectic markets
    AddFilter(SPF_SPREAD, MaxSpread);
    AddFilter(SPF_CHAOTIC_MARKET, true);
    
    return(INIT_SUCCEEDED);
}

//+------------------------------------------------------------------+
//| Expert tick function                                             |
//+------------------------------------------------------------------+
// This function is called on every tick of the selected currency pairs
void OnTick()
{
    for(int i=0; i<ArraySize(CurrencyPairs); i++)
    {
        // Check if currency pair is tradable
        if(IsTradable(CurrencyPairs[i]))
        {
            // Calculate mean reversion levels
            double mean = iMA(NULL, 0, 14, 0, MODE_SMA, PRICE_CLOSE, i);
            double upperLevel = mean + iATR(NULL, 0, 14, i) * 2;
            double lowerLevel = mean - iATR(NULL, 0, 14, i) * 2;
            
            // Check if price is above upper level, enter short trade
            if(Bid >= upperLevel)
            {
                double stopLoss = Ask + (Ask - Bid) * StopLossPercentage / 100;
                double takeProfit = Ask - (Ask - Bid) * TakeProfitPercentage / 100;
                double trailingStop = (Ask - Bid) * TrailingStopPercentage / 100;

                // Open short position
                int ticket = OrderSend(CurrencyPairs[i], OP_SELL, 0.01, Ask, 0, stopLoss, takeProfit, 'Red Hawk MT5', MagicNumber(), 0, Red);
                
                // Set trailing stop
                if(ticket > 0)
                    OrderModify(ticket, 0, stopLoss, takeProfit, 0, Red, trailingStop);
            }
            
            // Check if price is below lower level, enter long trade
            else if(Ask <= lowerLevel)
            {
                double stopLoss = Bid - (Ask - Bid) * StopLossPercentage / 100;
                double takeProfit = Bid + (Ask - Bid) * TakeProfitPercentage / 100;
                double trailingStop = (Ask - Bid) * TrailingStopPercentage / 100;

                // Open long position
                int ticket = OrderSend(CurrencyPairs[i], OP_BUY, 0.01, Bid, 0, stopLoss, takeProfit, 'Red Hawk MT5', MagicNumber(), 0, Green);
                
                // Set trailing stop
                if(ticket > 0)
                    OrderModify(ticket, 0, stopLoss, takeProfit, 0, Green, trailingStop);
            }
        }
    }
}

//+------------------------------------------------------------------+
//| Custom functions                                                 |
//+------------------------------------------------------------------+
// This function checks if the symbol is allowed for trading
bool IsTradable(string symbol)
{
    // Check if the symbol is allowed for trading
    for(int i=0; i<OrdersTotal(); i++)
    {
        if(OrderSelect(i, SELECT_BY_POS, MODE_TRADES))
        {
            if(OrderSymbol() == symbol)
                return false;
        }
    }
    
    return true;
}

// This function generates a unique magic number for each trade
int MagicNumber()
{
    int number = MathMod(TimeCurrent(), 1000000);
    return number;
}

// For detailed reviews and trading results of this product - https://forexroboteasy.com/forex-robot-review/red-hawk-mt5-review-efficient-forex-software-deal/
// ForexRobotEasy is not the official developer of this product. We only show sample code that can work as described in this product. 
// To find the official developer of this product use MQL5.
