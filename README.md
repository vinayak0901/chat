
Ceate a method in java.
which will give me near leg settlement date and far leg settlement date in response.

in method params i will provide -
currency 1
currency 2
transaction Date T
T + near leg days
T + far leg days

so you will have to call the below api for [T + near leg days] and [T + far leg days] seperatly 
and repeatadely by adding 1 day in the date
until it gets true in response for both the dates.
and you can return me near leg settlement date and far leg settlement date.

i have third party api, i will have to call this api to check the holidays
which return transactionAllowed (true or false) - 
request body of that api will contain -
currency 1
currency 2
transaction Date T

there will be an exception if currency 1 is USD and currency 2 is INR 
what ever the far leg settlement date you calculate normally - 
give me the month end date of that particular month 
also check the holiday for that month date and give me next




__________________________________________________________________________________________

in response im not getting bid_swap and ask_swap for EUR/USD directly.
im receiving following values

SWM_BIDPX
SWM_BIDPX2
SWM_BIDSPOTRATE
SWM_MIDPX
SWM_MIDPX2
SWM_MIDSPOTRATE
SWM_OFFERPX
SWM_OFFERPX2
SWM_OFFERSPOTRATE

0.0062939	
0.00629347	
0.0062939
0.0062937	
0.00629368	
0.0062937
0.0062935	
0.0062939	
0.0062935

how can i calculate the bid_swap and ask_swap?



________________________________________________________

FIX protocol.
I am sending a QuoteRequest[R] for Swap Points to be used 
in calculating FX rates for TD, TM, SP, Forward
with following params.
QuoteReqID=123XYZ
ProductType=FX-STD
Account=MyOrgName
Symbol=EUR/USD
Currency=EUR
QuoteType=1 (tradeable)
OrderQty=1000000 (Orderquantity(forswapsofnearleg). If market
data is requested, send‘0’)
OrderQty2=1000000 (Order quantity of far leg for swaps.Mandatory if
tradeable quotes for swaps are requested.)
SettlDate=(near leg settlement date)
SettlDate2=(far leg settlement date)

i have doubt if i should send 
RefSpotDate= (Defines the spot date in the RateProvider financial calender.
This value is used to clarify if both sides have the same
definition for a spot.)
Side=(Defines if theMarketTaker is intending tobuy
orsell thegivensymbol forFXandBaseMetals
products.
• TheSide<54>fielddefines thesideof the
firstcurrencyintheSymbol<55>field(the
basecurrency)andnotof thenotionalcur
rency.
• For FX Swap and NDS products, the
Side<54>fieldindicatesthesideof thefar
leg.
• IftheSide<54>fieldisnotpresent,thenthe
requestwillbeforatwo-sidedquote.
ForMoneyMarket(MM)products, theSide<54>
fieldismandatoryanddefinesif theclientwants
tolendorborrowmoney.
Possiblevalues:
• ‘1’=Buy
• ‘2’=Sell
• ‘F’=Lend(MMDepositonly)
• ‘G’=Borrow(MMLoanonly)
Note:
• For FX and NDF Block Trades, the
Side<54>fieldismandatoryanditsvalue
mustmatchthesideofthenettedlegquan
tities (defined using LegSide<624> and
LegQty<687>).
• Forzero-nettedblocktrades,thesideofthe
netoftheleg(s)withthenearesttenormust
beused(orbuyifthisisalsozero).)
