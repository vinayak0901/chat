package com.example.fx.service;

import com.example.fx.dto.SettlementDates;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestTemplate;

import java.time.LocalDate;
import java.time.temporal.TemporalAdjusters;

@Service
public class SettlementDateService {

    private final RestTemplate restTemplate;

    private final String holidayApiUrl =
            "https://third-party-api.com/holiday/check";

    public SettlementDateService(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }

    /**
     * Calculates near-leg and far-leg settlement dates.
     *
     * @param currency1       First currency
     * @param currency2       Second currency
     * @param transactionDate Transaction date T
     * @param nearLegDays     Near leg days
     * @param farLegDays      Far leg days
     * @return SettlementDates containing near and far settlement dates
     */
    public SettlementDates calculateSettlementDates(
            String currency1,
            String currency2,
            LocalDate transactionDate,
            int nearLegDays,
            int farLegDays) {

        if (transactionDate == null) {
            throw new IllegalArgumentException(
                    "Transaction date cannot be null");
        }

        if (nearLegDays < 0) {
            throw new IllegalArgumentException(
                    "Near leg days cannot be negative");
        }

        if (farLegDays < 0) {
            throw new IllegalArgumentException(
                    "Far leg days cannot be negative");
        }

        // ---------------------------------------------------------
        // Near Leg
        // ---------------------------------------------------------

        LocalDate nearLegCandidate =
                transactionDate.plusDays(nearLegDays);

        LocalDate nearLegSettlementDate =
                getNextValidSettlementDate(
                        currency1,
                        currency2,
                        nearLegCandidate
                );

        // ---------------------------------------------------------
        // Far Leg - Normal Calculation
        // ---------------------------------------------------------

        LocalDate farLegCandidate =
                transactionDate.plusDays(farLegDays);

        LocalDate farLegSettlementDate =
                getNextValidSettlementDate(
                        currency1,
                        currency2,
                        farLegCandidate
                );

        // ---------------------------------------------------------
        // USD/INR Special Handling
        // ---------------------------------------------------------

        if (isUsdInr(currency1, currency2)) {

            /*
             * Take the last day of the month in which the
             * normally calculated far-leg date falls.
             */
            LocalDate monthEnd =
                    farLegSettlementDate
                            .with(TemporalAdjusters.lastDayOfMonth());

            /*
             * Check month-end.
             *
             * If it is a holiday, keep adding one day until
             * transactionAllowed = true.
             */
            farLegSettlementDate =
                    getNextValidSettlementDate(
                            currency1,
                            currency2,
                            monthEnd
                    );
        }

        return new SettlementDates(
                nearLegSettlementDate,
                farLegSettlementDate
        );
    }

    /**
     * Starting from candidateDate, repeatedly calls the
     * third-party API until transactionAllowed = true.
     *
     * There is intentionally NO maximum attempt limit.
     */
    private LocalDate getNextValidSettlementDate(
            String currency1,
            String currency2,
            LocalDate candidateDate) {

        LocalDate date = candidateDate;

        while (true) {

            boolean transactionAllowed =
                    checkTransactionAllowed(
                            currency1,
                            currency2,
                            date
                    );

            if (transactionAllowed) {
                return date;
            }

            date = date.plusDays(1);
        }
    }

    /**
     * Calls the third-party holiday/calendar API.
     */
    private boolean checkTransactionAllowed(
            String currency1,
            String currency2,
            LocalDate transactionDate) {

        HolidayCheckRequest request =
                new HolidayCheckRequest(
                        currency1,
                        currency2,
                        transactionDate
                );

        HolidayCheckResponse response =
                restTemplate.postForObject(
                        holidayApiUrl,
                        request,
                        HolidayCheckResponse.class
                );

        if (response == null) {
            throw new IllegalStateException(
                    "Empty response received from holiday API"
            );
        }

        return response.isTransactionAllowed();
    }

    /**
     * Checks specifically for USD/INR.
     */
    private boolean isUsdInr(
            String currency1,
            String currency2) {

        return "USD".equalsIgnoreCase(currency1)
                && "INR".equalsIgnoreCase(currency2);
    }
}

_______________
package com.example.fx.service;

import java.time.LocalDate;

public class HolidayCheckRequest {

    private String currency1;
    private String currency2;
    private LocalDate transactionDate;

    public HolidayCheckRequest() {
    }

    public HolidayCheckRequest(
            String currency1,
            String currency2,
            LocalDate transactionDate) {

        this.currency1 = currency1;
        this.currency2 = currency2;
        this.transactionDate = transactionDate;
    }

    public String getCurrency1() {
        return currency1;
    }

    public void setCurrency1(String currency1) {
        this.currency1 = currency1;
    }

    public String getCurrency2() {
        return currency2;
    }

    public void setCurrency2(String currency2) {
        this.currency2 = currency2;
    }

    public LocalDate getTransactionDate() {
        return transactionDate;
    }

    public void setTransactionDate(LocalDate transactionDate) {
        this.transactionDate = transactionDate;
    }
}
_____________
package com.example.fx.service;

public class HolidayCheckResponse {

    private boolean transactionAllowed;

    public HolidayCheckResponse() {
    }

    public boolean isTransactionAllowed() {
        return transactionAllowed;
    }

    public void setTransactionAllowed(
            boolean transactionAllowed) {

        this.transactionAllowed = transactionAllowed;
    }
}
_______________
package com.example.fx.dto;

import java.time.LocalDate;

public class SettlementDates {

    private final LocalDate nearLegSettlementDate;
    private final LocalDate farLegSettlementDate;

    public SettlementDates(
            LocalDate nearLegSettlementDate,
            LocalDate farLegSettlementDate) {

        this.nearLegSettlementDate =
                nearLegSettlementDate;

        this.farLegSettlementDate =
                farLegSettlementDate;
    }

    public LocalDate getNearLegSettlementDate() {
        return nearLegSettlementDate;
    }

    public LocalDate getFarLegSettlementDate() {
        return farLegSettlementDate;
    }
}
________________

package com.example.fx.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

@Configuration
public class RestTemplateConfig {

    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
__________________


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
