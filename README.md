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
