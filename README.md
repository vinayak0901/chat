public Quote getBestQuote(String quoteRequestId) {

    List<Quote> quotes = quoteMap.get(quoteRequestId);

    if (quotes == null || quotes.isEmpty()) {
        return null;
    }

    return quotes.stream()
            .max(Comparator.comparing(Quote::getBidPx)
                    .thenComparing(Quote::getOfferPx))
            .orElse(null);
}

__________


package com.example.service;

import org.springframework.stereotype.Component;
import java.util.Set;
import java.util.concurrent.ConcurrentHashMap;

@Component
public class DynamicSetManager {

    // Thread-safe set designed for concurrent reads/writes
    private final Set<String> sharedSet = ConcurrentHashMap.newKeySet();

    public boolean add(String item) {
        return sharedSet.add(item);
    }

    public boolean remove(String item) {
        return sharedSet.remove(item);
    }

    public boolean contains(String item) {
        return sharedSet.contains(item);
    }

    public Set<String> getAll() {
        return Set.copyOf(sharedSet); // Returns a read-only snapshot
    }
}
____________

package com.example.fx.service;

import com.example.fx.dto.DateCalculationType;
import com.example.fx.dto.LegDateRule;
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
     * @param currency1
     * @param currency2
     * @param transactionDate Transaction date T
     * @param nearLeg         Near-leg date calculation rule
     * @param farLeg          Far-leg date calculation rule
     */
    public SettlementDates calculateSettlementDates(
            String currency1,
            String currency2,
            LocalDate transactionDate,
            LegDateRule nearLeg,
            LegDateRule farLeg) {

        if (transactionDate == null) {
            throw new IllegalArgumentException(
                    "Transaction date cannot be null");
        }

        if (nearLeg == null) {
            throw new IllegalArgumentException(
                    "Near leg rule cannot be null");
        }

        if (farLeg == null) {
            throw new IllegalArgumentException(
                    "Far leg rule cannot be null");
        }

        // ---------------------------------------------------------
        // 1. Find the first valid transaction date
        // ---------------------------------------------------------

        LocalDate validTransactionDate =
                getNextValidDate(
                        currency1,
                        currency2,
                        transactionDate
                );

        // ---------------------------------------------------------
        // 2. Calculate Near Leg
        // ---------------------------------------------------------

        LocalDate nearLegSettlementDate =
                calculateLegDate(
                        currency1,
                        currency2,
                        validTransactionDate,
                        nearLeg
                );

        // ---------------------------------------------------------
        // 3. Calculate Far Leg
        // ---------------------------------------------------------

        /*
         * CALENDAR_DAYS_ROLL / Forward calculation is based
         * on the NEAR LEG settlement date.
         *
         * BUSINESS_DAYS calculation is based on the valid
         * transaction date.
         */
        LocalDate farLegStartDate;

        if (farLeg.getType()
                == DateCalculationType.CALENDAR_DAYS_ROLL) {

            farLegStartDate = nearLegSettlementDate;

        } else {

            farLegStartDate = validTransactionDate;
        }

        LocalDate farLegSettlementDate =
                calculateLegDate(
                        currency1,
                        currency2,
                        farLegStartDate,
                        farLeg
                );

        // ---------------------------------------------------------
        // 4. USD/INR special handling
        // ---------------------------------------------------------

        if (isUsdInr(currency1, currency2)) {

            /*
             * Take the last day of the month in which the
             * calculated far-leg date falls.
             */
            LocalDate monthEnd =
                    farLegSettlementDate
                            .with(
                                    TemporalAdjusters
                                            .lastDayOfMonth()
                            );

            /*
             * If month-end is not allowed, move forward
             * until transactionAllowed = true.
             */
            farLegSettlementDate =
                    getNextValidDate(
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
     * Calculates a leg date based on the requested rule.
     */
    private LocalDate calculateLegDate(
            String currency1,
            String currency2,
            LocalDate startDate,
            LegDateRule legRule) {

        DateCalculationType type =
                legRule.getType();

        int days =
                legRule.getDays();

        switch (type) {

            case BUSINESS_DAYS:

                return addBusinessDays(
                        currency1,
                        currency2,
                        startDate,
                        days
                );

            case CALENDAR_DAYS_ROLL:

                return addCalendarDaysAndRoll(
                        currency1,
                        currency2,
                        startDate,
                        days
                );

            default:

                throw new IllegalArgumentException(
                        "Unsupported date calculation type: "
                                + type
                );
        }
    }

    /**
     * BUSINESS_DAYS calculation.
     *
     * Only dates where transactionAllowed = true
     * are counted.
     *
     * Example:
     *
     * Start = 9
     * Days = 2
     *
     * 10 -> false
     * 11 -> false
     * 12 -> true  = day 1
     * 13 -> true  = day 2
     *
     * Result = 13
     */
    private LocalDate addBusinessDays(
            String currency1,
            String currency2,
            LocalDate startDate,
            int daysToAdd) {

        LocalDate date = startDate;

        int validDaysAdded = 0;

        while (validDaysAdded < daysToAdd) {

            date = date.plusDays(1);

            boolean transactionAllowed =
                    checkTransactionAllowed(
                            currency1,
                            currency2,
                            date
                    );

            if (transactionAllowed) {
                validDaysAdded++;
            }
        }

        return date;
    }

    /**
     * CALENDAR_DAYS_ROLL calculation.
     *
     * First add the specified number of calendar days.
     * Then check the resulting date.
     *
     * If it is not allowed, move forward one day at a time
     * until transactionAllowed = true.
     *
     * Example:
     *
     * Start = 9
     * Days = 2
     *
     * 9 + 2 = 11
     *
     * 11 -> false
     * 12 -> false
     * 13 -> true
     *
     * Result = 13
     */
    private LocalDate addCalendarDaysAndRoll(
            String currency1,
            String currency2,
            LocalDate startDate,
            int daysToAdd) {

        LocalDate date =
                startDate.plusDays(daysToAdd);

        return getNextValidDate(
                currency1,
                currency2,
                date
        );
    }

    /**
     * Finds the first valid date starting from candidateDate.
     *
     * The candidate date itself is checked first.
     */
    private LocalDate getNextValidDate(
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
     * Calls the third-party holiday API.
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

    private boolean isUsdInr(
            String currency1,
            String currency2) {

        return "USD".equalsIgnoreCase(currency1)
                && "INR".equalsIgnoreCase(currency2);
    }
}

__________________

package com.example.fx.dto;

public class LegDateRule {

    private final int days;
    private final DateCalculationType type;

    public LegDateRule(
            int days,
            DateCalculationType type) {

        if (days < 0) {
            throw new IllegalArgumentException(
                    "Days cannot be negative");
        }

        if (type == null) {
            throw new IllegalArgumentException(
                    "Date calculation type cannot be null");
        }

        this.days = days;
        this.type = type;
    }

    public int getDays() {
        return days;
    }

    public DateCalculationType getType() {
        return type;
    }
}

_______________

package com.example.fx.dto;

public enum DateCalculationType {

    /**
     * Only valid transaction dates are counted.
     *
     * Example:
     * Start = 9
     * Days = 2
     *
     * 10 = false
     * 11 = false
     * 12 = true   -> day 1
     * 13 = true   -> day 2
     *
     * Result = 13
     */
    BUSINESS_DAYS,

    /**
     * Add calendar days first, then check the resulting date.
     * If the resulting date is invalid, move forward one day
     * at a time until a valid date is found.
     *
     * Example:
     * Start = 9
     * Days = 2
     *
     * 11 = false
     * 12 = false
     * 13 = true
     *
     * Result = 13
     */
    CALENDAR_DAYS_ROLL
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
