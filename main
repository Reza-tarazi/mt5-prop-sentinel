//+------------------------------------------------------------------+
//|                                     PropDisciplineSentinel.mq5   |
//|    Real-Time Prop Discipline, On-Chart PopUp Protocol & HUD      |
//|               Author: Reza Tarazi | Trading Behavioral Engine    |
//+------------------------------------------------------------------+
#property copyright "Trading Discipline System"
#property link      "https://github.com/Reza-tarazi/trade-behavioral-analytics"
#property version   "8.00"
#property strict

//--- تنظیمات تلگرام
input group "=== Telegram Settings ==="
input string   InpTelegramBotToken   = "YOUR_BOT_TOKEN";    // توکن ربات تلگرام
input string   InpTelegramChatID     = "YOUR_CHAT_ID";      // شناسه چت یا کانال تلگرام

//--- تنظیمات انضباطی و ریسک روزانه
input group "=== Dynamic Risk & Discipline Rules ==="
input int      InpMaxConsecutiveLosses = 2;       // سقف مجاز ضررهای متوالی روزانه
input int      InpCooldownMinutes      = 10;      // وقفه اجباری پس از هر باخت (دقیقه)
input double   InpDailyTargetProfitPct = 2.0;     // سقف سود روزانه برای توقف ترید (درصد)
input double   InpMaxDailyRiskPct      = 1.0;     // حداکثر افت مجاز روزانه (درصد)
input double   InpRiskPerTradePct      = 0.5;     // ریسک استاندارد هر پوزیشن (درصد)
input bool     InpEnableEntryPopup     = true;    // نمایش پاپ‌آپ پروتکل تعهد آگاهانه هنگام ورود

//--- ساختار نگهداری وضعیت پوزیشن‌ها
struct PosMemory
{
   ulong  ticket;
   double sl;
   double open_price;
   long   type;
   string symbol;
   double volume;
};

PosMemory tracked_list[];

datetime current_day;
double   day_start_balance    = 0.0;
int      consecutive_losses   = 0;
bool     daily_target_hit     = false;
bool     daily_loss_hit       = false;
datetime last_loss_close_time = 0;
int      atr_handle           = INVALID_HANDLE;

//+------------------------------------------------------------------+
//| ارسال پیام به تلگرام با فرمت امن HTML                           |
//+------------------------------------------------------------------+
bool SendTelegramMessage(string message)
{
   if(InpTelegramBotToken == "YOUR_BOT_TOKEN" || InpTelegramChatID == "YOUR_CHAT_ID" || InpTelegramBotToken == "")
      return false;

   string url = "https://api.telegram.org/bot" + InpTelegramBotToken + "/sendMessage";
   StringReplace(message, "\"", "\\\"");
   string json_payload = "{\"chat_id\":\"" + InpTelegramChatID + "\", \"text\":\"" + message + "\", \"parse_mode\":\"HTML\"}";
   
   char data[];
   StringToCharArray(json_payload, data, 0, WHOLE_ARRAY, CP_UTF8);
   ArrayResize(data, ArraySize(data) - 1);
   
   string headers = "Content-Type: application/json\r\n";
   char result[];
   string result_headers;
   
   ResetLastError();
   int res = WebRequest("POST", url, headers, 5000, data, result, result_headers);
   return (res == 200);
}

//+------------------------------------------------------------------+
//| تشخیص سشن جاری بازار                                            |
//+------------------------------------------------------------------+
string GetMarketSession()
{
   MqlDateTime dt;
   TimeCurrent(dt);
   if(dt.hour >= 0 && dt.hour < 8)   return "آسیا (Asia) 🌙";
   if(dt.hour >= 8 && dt.hour < 16)  return "لندن (London) 🟢";
   return "نیویورک (New York) 🏛";
}

//+------------------------------------------------------------------+
//| ترسیم پنل هوشمند اطلاعات ورود روی چارت                           |
//+------------------------------------------------------------------+
void UpdateChartHUD()
{
   double atr_val = 0.0;
   if(atr_handle != INVALID_HANDLE)
   {
      double atr_buf[];
      ArraySetAsSeries(atr_buf, true);
      if(CopyBuffer(atr_handle, 0, 0, 1, atr_buf) > 0)
         atr_val = atr_buf[0];
   }

   double risk_usd = day_start_balance * (InpRiskPerTradePct / 100.0);
   double safe_sl_distance = (atr_val > 0) ? (atr_val * 1.2) : 10.0;

   double contract_size = SymbolInfoDouble(_Symbol, SYMBOL_TRADE_CONTRACT_SIZE);
   if(contract_size <= 0) contract_size = 100.0;

   double suggested_lot = 0.01;
   if(safe_sl_distance > 0 && contract_size > 0)
   {
      suggested_lot = risk_usd / (safe_sl_distance * contract_size);
      double min_lot  = SymbolInfoDouble(_Symbol, SYMBOL_VOLUME_MIN);
      double max_lot  = SymbolInfoDouble(_Symbol, SYMBOL_VOLUME_MAX);
      double step_lot = SymbolInfoDouble(_Symbol, SYMBOL_VOLUME_STEP);
      
      suggested_lot = MathFloor(suggested_lot / step_lot) * step_lot;
      if(suggested_lot < min_lot) suggested_lot = min_lot;
      if(suggested_lot > max_lot) suggested_lot = max_lot;
   }

   string hud = "═════════ 🛡 دیدبان ورود آگاهانه (Prop HUD) ═════════\n" +
                "• سشن جاری بازار: " + GetMarketSession() + "\n" +
                "• نوسان لحظه‌ای (ATR): " + DoubleToString(atr_val, 2) + " $\n" +
                "• حداقل فاصله امن استاپ (1.2x ATR): " + DoubleToString(safe_sl_distance, 2) + " $\n" +
                "• سقف ریسک معامله (0.5%): $" + DoubleToString(risk_usd, 2) + "\n" +
                "• لات دقیق پیشنهادی (ریسک ~$52): " + DoubleToString(suggested_lot, 2) + " Lot\n" +
                "• وضعیت امروز: " + IntegerToString(consecutive_losses) + " باخت از " + IntegerToString(InpMaxConsecutiveLosses) + " مجاز\n" +
                "══════════════════════════════════════════";

   Comment(hud);
}

//+------------------------------------------------------------------+
//| مقداردهی اولیه                                                   |
//+------------------------------------------------------------------+
int OnInit()
{
   MqlDateTime dt;
   TimeCurrent(dt);
   dt.hour = 0; dt.min = 0; dt.sec = 0;
   current_day = StructToTime(dt);
   day_start_balance = AccountInfoDouble(ACCOUNT_BALANCE);

   atr_handle = iATR(_Symbol, _Period, 14);

   SendTelegramMessage("🛡 <b>دیدبان انضباط معاملاتی فعال شد</b>\n" +
                       "• بالانس شروع: $" + DoubleToString(day_start_balance, 2) + "\n" +
                       "• پاپ‌آپ پروتکل تعهد آگاهانه متاتریدر: <b>فعال</b>");
   return(INIT_SUCCEEDED);
}

void OnDeinit(const int reason)
{
   Comment("");
   if(atr_handle != INVALID_HANDLE)
      IndicatorRelease(atr_handle);
}

//+------------------------------------------------------------------+
//| کشف تغییرات استاپ پوزیشن‌ها                                       |
//+------------------------------------------------------------------+
void CheckInstantSLChange()
{
   int total = PositionsTotal();
   for(int i = 0; i < total; i++)
   {
      ulong ticket = PositionGetTicket(i);
      if(!PositionSelectByTicket(ticket)) continue;

      double cur_sl     = PositionGetDouble(POSITION_SL);
      double open_price = PositionGetDouble(POSITION_PRICE_OPEN);
      long   pos_type   = PositionGetInteger(POSITION_TYPE);
      string symbol     = PositionGetString(POSITION_SYMBOL);
      double volume     = PositionGetDouble(POSITION_VOLUME);

      int idx = -1;
      for(int j = 0; j < ArraySize(tracked_list); j++)
      {
         if(tracked_list[j].ticket == ticket) { idx = j; break; }
      }

      if(idx == -1)
      {
         int sz = ArraySize(tracked_list);
         ArrayResize(tracked_list, sz + 1);
         tracked_list[sz].ticket = ticket;
         tracked_list[sz].sl = cur_sl;
         tracked_list[sz].open_price = open_price;
         tracked_list[sz].type = pos_type;
         tracked_list[sz].symbol = symbol;
         tracked_list[sz].volume = volume;
         continue;
      }

      double old_sl = tracked_list[idx].sl;
      if(cur_sl != old_sl)
      {
         int digits = (int)SymbolInfoInteger(symbol, SYMBOL_DIGITS);
         if(cur_sl == 0.0 && old_sl > 0.0)
         {
            SendTelegramMessage("🚨 <b>اخطار بحرانی: حد ضرر کاملاً حذف شد!</b>\n• معامله #" + IntegerToString(ticket) + " (" + symbol + ")");
         }
         else if(old_sl > 0.0 && cur_sl > 0.0)
         {
            bool risk_increased = (pos_type == POSITION_TYPE_BUY && cur_sl < old_sl) || (pos_type == POSITION_TYPE_SELL && cur_sl > old_sl);
            if(risk_increased)
            {
               double tick_size = SymbolInfoDouble(symbol, SYMBOL_TRADE_TICK_SIZE);
               double tick_val  = SymbolInfoDouble(symbol, SYMBOL_TRADE_TICK_VALUE);
               if(tick_size <= 0) tick_size = _Point;
               double added_risk = (MathAbs(cur_sl - old_sl) / tick_size) * tick_val * volume;

               SendTelegramMessage("⚠️ <b>هشدار رفتاری: عقب کشیدن استاپ</b>\n" +
                                   "• معامله #" + IntegerToString(ticket) + "\n" +
                                   "• استاپ قبلی: " + DoubleToString(old_sl, digits) + "\n" +
                                   "• استاپ جدید: " + DoubleToString(cur_sl, digits) + "\n" +
                                   "• ریسک اضافه تحمیلی: ~$" + DoubleToString(added_risk, 2));
            }
            else
            {
               bool is_be = (pos_type == POSITION_TYPE_BUY && cur_sl >= open_price) || (pos_type == POSITION_TYPE_SELL && cur_sl <= open_price);
               if(is_be)
                  SendTelegramMessage("🛡 <b>انضباط معاملاتی:</b> معامله #" + IntegerToString(ticket) + " (" + symbol + ") ریسک‌فری شد.");
            }
         }
         tracked_list[idx].sl = cur_sl;
      }
   }
}

//+------------------------------------------------------------------+
//| رویداد تراکنش‌های ورود، خروج و توالی باخت                        |
//+------------------------------------------------------------------+
void OnTradeTransaction(const MqlTradeTransaction &trans,
                         const MqlTradeRequest &request,
                         const MqlTradeResult &result)
{
   CheckInstantSLChange();

   if(trans.type == TRADE_TRANSACTION_DEAL_ADD)
   {
      ulong deal_ticket = trans.deal;
      if(HistoryDealSelect(deal_ticket))
      {
         ENUM_DEAL_ENTRY entry = (ENUM_DEAL_ENTRY)HistoryDealGetInteger(deal_ticket, DEAL_ENTRY);
         string symbol         = HistoryDealGetString(deal_ticket, DEAL_SYMBOL);
         datetime deal_time    = (datetime)HistoryDealGetInteger(deal_ticket, DEAL_TIME);

         // الف) باز شدن پوزیشن جدید (لحظه کلیک روی Buy یا Sell)
         if(entry == DEAL_ENTRY_IN)
         {
            // ۱. نمایش پاپ‌آپ پروتکل تعهد آگاهانه روی صفحه متاتریدر
            if(InpEnableEntryPopup)
            {
               string popup_msg = "⚠️ [پروتکل تعهد آگاهانه - ۳۰ ثانیه تامل]\n" +
                                  "معامله روی نماد " + symbol + " باز شد.\n\n" +
                                  "۱. آیا ورود بر اساس پلن است و شتاب‌زدگی نیست؟\n" +
                                  "۲. آیا ریسک معامله دقیقاً ۰.۵٪ بالانس است؟\n" +
                                  "۳. تعهد: تا انتهای پوزیشن، استاپ به عقب کشیده نمی‌شود!\n";
               
               if(consecutive_losses >= InpMaxConsecutiveLosses)
                  popup_msg += "\n🚨 اخطار بحرانی: سقف باخت‌های امروز پر است!";
               
               Alert(popup_msg);
            }

            // ۲. بررسی نقض سقف ضررهای متوالی
            if(consecutive_losses >= InpMaxConsecutiveLosses)
            {
               SendTelegramMessage("⛔️ <b>اخطار جدی: معامله خلاف توقف روزانه!</b>\n" +
                                   "• نماد: " + symbol + "\n" +
                                   "• سقف باخت‌های امروز پر شده بود؛ ورود ممنوع بوده است.");
            }

            // ۳. بررسی نقض زمان وقفه بعد از ضرر
            if(last_loss_close_time > 0)
            {
               long sec_passed = (long)(deal_time - last_loss_close_time);
               if(sec_passed < InpCooldownMinutes * 60)
               {
                  SendTelegramMessage("🔥 <b>هشدار ترید شتاب‌زده (Revenge Alert):</b>\n" +
                                      "• نماد: " + symbol + "\n" +
                                      "• تنها " + IntegerToString(sec_passed / 60) + " دقیقه از ضرر قبلی گذشته است (وقفه الزامی: " + IntegerToString(InpCooldownMinutes) + " دقیقه).");
               }
            }
         }

         // ب) بسته شدن پوزیشن
         if(entry == DEAL_ENTRY_OUT)
         {
            double net = HistoryDealGetDouble(deal_ticket, DEAL_PROFIT) +
                         HistoryDealGetDouble(deal_ticket, DEAL_COMMISSION) +
                         HistoryDealGetDouble(deal_ticket, DEAL_SWAP);

            if(net < 0.0)
            {
               consecutive_losses++;
               last_loss_close_time = deal_time;
               string msg = "❌ <b>معامله با ضرر بسته شد:</b> " + symbol + "\n" +
                            "• ضرر خالص: $" + DoubleToString(net, 2) + "\n" +
                            "• ضررهای متوالی امروز: " + IntegerToString(consecutive_losses) + " از " + IntegerToString(InpMaxConsecutiveLosses) + "\n" +
                            "• ⏱ <b>شروع وقفه:</b> تا " + IntegerToString(InpCooldownMinutes) + " دقیقه ورود بعدی ممنوع است.";
               if(consecutive_losses >= InpMaxConsecutiveLosses)
                  msg += "\n\n🛑 <b>توقف روزانه فعال شد.</b>";
               SendTelegramMessage(msg);
            }
            else if(net > 0.0)
            {
               consecutive_losses = 0;
               SendTelegramMessage("✅ <b>معامله سودده بسته شد:</b> " + symbol + " (+$" + DoubleToString(net, 2) + ")\n• زنجیره باخت‌ها صفر شد.");
            }
         }
      }
   }
}

//+------------------------------------------------------------------+
//| پایش تیک‌ها و به‌روزرسانی پنل چارت                               |
//+------------------------------------------------------------------+
void OnTick()
{
   CheckInstantSLChange();
   UpdateChartHUD();

   MqlDateTime dt;
   TimeCurrent(dt);
   dt.hour = 0; dt.min = 0; dt.sec = 0;
   datetime today_zero = StructToTime(dt);

   if(today_zero > current_day)
   {
      current_day = today_zero;
      day_start_balance    = AccountInfoDouble(ACCOUNT_BALANCE);
      consecutive_losses   = 0;
      daily_target_hit     = false;
      daily_loss_hit       = false;
      last_loss_close_time = 0;
      SendTelegramMessage("🔄 <b>روز جدید معاملاتی آغاز شد.</b>");
   }

   double equity = AccountInfoDouble(ACCOUNT_EQUITY);
   if(day_start_balance > 0.0)
   {
      double pnl_pct = ((equity - day_start_balance) / day_start_balance) * 100.0;
      if(pnl_pct >= InpDailyTargetProfitPct && !daily_target_hit)
      {
         daily_target_hit = true;
         SendTelegramMessage("🎯 <b>سقف سود روزانه محقق شد (+" + DoubleToString(pnl_pct, 2) + "%).</b> پلتفرم را ببندید.");
      }
      if(pnl_pct <= -InpMaxDailyRiskPct && !daily_loss_hit)
      {
         daily_loss_hit = true;
         SendTelegramMessage("🚨 <b>سقف افت روزانه لمس شد.</b> خروج فوری الزامی است.");
      }
   }
}
//+------------------------------------------------------------------+
