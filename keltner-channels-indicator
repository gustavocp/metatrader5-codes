//+------------------------------------------------------------------+
//|                                                   KeltnerChannel.mq5 |
//|                        Copyright 2023, MetaQuotes Software Corp. |
//|                                        https://www.mql5.com      |
//+------------------------------------------------------------------+
#property indicator_chart_window
#property indicator_buffers 5
#property indicator_plots   5

#property indicator_label1  "Keltner Up"
#property indicator_type1   DRAW_LINE
#property indicator_color1  clrRed
#property indicator_style1  STYLE_SOLID
#property indicator_width1  1

#property indicator_label2  "Keltner Down"
#property indicator_type2   DRAW_LINE
#property indicator_color2  clrBlue
#property indicator_style2  STYLE_SOLID
#property indicator_width2  1

#property indicator_label3  "Keltner Mid"
#property indicator_type3   DRAW_LINE
#property indicator_color3  clrGreen
#property indicator_style3  STYLE_SOLID
#property indicator_width3  1

#property indicator_label4  "Keltner Stop Up"
#property indicator_type4   DRAW_LINE
#property indicator_color4  clrMagenta
#property indicator_style4  STYLE_DASH
#property indicator_width4  1

#property indicator_label5  "Keltner Stop Down"
#property indicator_type5   DRAW_LINE
#property indicator_color5  clrOrange
#property indicator_style5  STYLE_DASH
#property indicator_width5  1

#include <Trade\Trade.mqh>

//--- input parameters
input double atr_multiplier = 2.0;
input int atr_period = 20;
input int ema_period = 20;
input double lot_size = 0.1;

//--- indicator buffers
double KeltnerUp[];
double KeltnerDown[];
double KeltnerMid[];
double KeltnerStopUp[];
double KeltnerStopDown[];

//--- handles for the indicators
int handleATR;
int handleEMA;

CTrade trade;

//+------------------------------------------------------------------+
//| Custom indicator initialization function                         |
//+------------------------------------------------------------------+
int OnInit()
  {
   // Indicator buffers mapping
   SetIndexBuffer(0, KeltnerUp);
   SetIndexBuffer(1, KeltnerDown);
   SetIndexBuffer(2, KeltnerMid);
   SetIndexBuffer(3, KeltnerStopUp);
   SetIndexBuffer(4, KeltnerStopDown);

   // Set the arrays as series
   ArraySetAsSeries(KeltnerUp, true);
   ArraySetAsSeries(KeltnerDown, true);
   ArraySetAsSeries(KeltnerMid, true);
   ArraySetAsSeries(KeltnerStopUp, true);
   ArraySetAsSeries(KeltnerStopDown, true);

   // Create the handles for ATR and EMA
   handleATR = iATR(_Symbol, _Period, atr_period);
   handleEMA = iMA(_Symbol, _Period, ema_period, 0, MODE_EMA, PRICE_CLOSE);

   if (handleATR == INVALID_HANDLE || handleEMA == INVALID_HANDLE)
     {
      Print("Error creating indicator handles");
      return (INIT_FAILED);
     }

   return (INIT_SUCCEEDED);
  }
//+------------------------------------------------------------------+
//| Custom indicator deinitialization function                       |
//+------------------------------------------------------------------+
void OnDeinit(const int reason)
  {
   IndicatorRelease(handleATR);
   IndicatorRelease(handleEMA);
  }
//+------------------------------------------------------------------+
//| Custom indicator iteration function                              |
//+------------------------------------------------------------------+
int OnCalculate(const int rates_total,
                const int prev_calculated,
                const datetime &time[],
                const double &open[],
                const double &high[],
                const double &low[],
                const double &close[],
                const long &tick_volume[],
                const long &volume[],
                const int &spread[])
  {
   if (rates_total < atr_period || rates_total < ema_period)
     return (0);

   double atr[], ema[];
   ArraySetAsSeries(atr, true);
   ArraySetAsSeries(ema, true);

   if (CopyBuffer(handleATR, 0, 0, rates_total, atr) <= 0)
     return (0);
   if (CopyBuffer(handleEMA, 0, 0, rates_total, ema) <= 0)
     return (0);

   for (int i = 0; i < rates_total; i++)
     {
      double atr_i = atr[i];
      double ema_i = ema[i];

      KeltnerUp[i] = ema_i + atr_multiplier * atr_i;
      KeltnerDown[i] = ema_i - atr_multiplier * atr_i;
      KeltnerMid[i] = (KeltnerUp[i] + KeltnerDown[i]) / 2;
      KeltnerStopUp[i] = (KeltnerUp[i] + KeltnerMid[i]) / 2;
      KeltnerStopDown[i] = (KeltnerDown[i] + KeltnerMid[i]) / 2;
     }

   // Ensure lines are drawn correctly in backtest mode
   for (int i = prev_calculated; i < rates_total; i++)
     {
      KeltnerUp[i] = ema[i] + atr_multiplier * atr[i];
      KeltnerDown[i] = ema[i] - atr_multiplier * atr[i];
      KeltnerMid[i] = (KeltnerUp[i] + KeltnerDown[i]) / 2;
      KeltnerStopUp[i] = (KeltnerUp[i] + KeltnerMid[i]) / 2;
      KeltnerStopDown[i] = (KeltnerDown[i] + KeltnerMid[i]) / 2;
     }

   // Check trading conditions
   if (rates_total > 1 && prev_calculated > 0)
     {
      double prevClose = close[1];
      double currClose = close[0];

      Print("Previous Close: ", prevClose, " Current Close: ", currClose);
      Print("KeltnerUp[1]: ", KeltnerUp[1], " KeltnerUp[0]: ", KeltnerUp[0]);
      Print("KeltnerStopUp[1]: ", KeltnerStopUp[1], " KeltnerStopUp[0]: ", KeltnerStopUp[0]);
      Print("KeltnerDown[1]: ", KeltnerDown[1], " KeltnerDown[0]: ", KeltnerDown[0]);
      Print("KeltnerStopDown[1]: ", KeltnerStopDown[1], " KeltnerStopDown[0]: ", KeltnerStopDown[0]);

      // Sell condition
      if ((prevClose > KeltnerUp[1] && currClose < KeltnerUp[0]) || (prevClose > KeltnerStopUp[1] && currClose < KeltnerStopUp[0]))
        {
         Print("Sell Condition Met");
         if (trade.Sell(lot_size))
           {
            Print("Sell Order Placed");
           }
         else
           {
            Print("Sell Order Failed: ", GetLastError());
           }
        }
      // Buy condition
      else if ((prevClose < KeltnerDown[1] && currClose > KeltnerDown[0]) || (prevClose < KeltnerStopDown[1] && currClose > KeltnerStopDown[0]))
        {
         Print("Buy Condition Met");
         if (trade.Buy(lot_size))
           {
            Print("Buy Order Placed");
           }
         else
           {
            Print("Buy Order Failed: ", GetLastError());
           }
        }
     }

   return (rates_total);
  }
//+------------------------------------------------------------------+

