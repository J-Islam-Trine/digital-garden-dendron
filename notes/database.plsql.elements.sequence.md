
CREATE OR REPLACE 
PACKAGE BODY PCRD_STA_REPORTING_TOOLS_1_V3 IS
-------------------------------------------------------------------------------------------------------------------------------
--  VERSION      DATE             AUTHOR               REF           REMARKS
-------------------------------------------------------------------------------------------------------------------------------
-- V3.0.1        10/03/2015       B.SAAD                             INITIAL VERSION - STA-03 -  VISA QUATERLY CREDIT REPORT
-- V3.0.2        24/04/2015       B.SAAD                             IMPL�MENTATION CCR 13 SAB24042015
-- V3.0.1        17/06/2015       B.SAAD                             INITIAL VERSION - STA-07 -  BRANCH CARDS BLOCKED
-- V3.0.2        30/06/2015       B.SAAD                             FIX PROD00018655  SEE SAB30062015
-- V3.0.3        16/07/2015       B.SAAD                             FIX CLASS_LEVEL  SEE SAB16072015
-- V3.0.4        06/10/2015       B.SAAD                             FIX ATM TRX  SEE SAB06102015
-- V3.0.5        27/10/2015       B.SAAD                             CALCULATE DELINQUENCY SECTION DWH SEE SAB27102015
-- V3.0.6        07/04/2016       M.BENHAMMOU                        CASE OF ACTION_CODE IS NULL SEE MBH07042016
-- V3.0.7        02/06/2016       T.EDDEBBAGH                        SEE TED02062016 - ADD ORIGINE CODE TO DISPLAY NATIONAL TRX
-- V3.0.8        11/08/2016       T.EDDEBBAGH                        SEE TED20160811 - DISPLAY TRX WITH CARD_TYPE NULL IN THE REPORT
-- V3.0.9        21/04/2017       M.BENHAMMOU                        ADD MULTIBANKING ASPECT TO REPORT  --MBH21042017
-- V3.1.0        14/11/2017       M.BENHAMMOU                        DROP TABLE STG_BRANCH_CARDS_BLOCKED_HIST
-- V3.1.1        10/01/2018       M.BENHAMMOU        MBH10012018     NEW STATISTIC BATCH PARAMETERS BY BANK/NETWORK/CURRENCY
--(NOTE: THE AMOUNT_LOCAL_CURRENCY FIELDS IN SOME STAGING TABLES MAY CONTAIN VALUES WITH THE CURRENCY USER_CURRENCY. (TO NOT IMPACT THE JASPERS)
-- V3.1.2        16/11/2022       FZ.HANIN                           SEE HFZ20221114 , PLUTISS-11651-ENH22Q4 -ARTICLE-8.5.1
--------------------------------------------------------------------------------------------------------------------------------

V_ENV_INFO_TRACE                GLOBAL_VARS.ENV_INFO_TRACE_TYPE;

PROCEDURE   VERSION IS
BEGIN
    PCRD_GENERAL_TOOLS_4.GET_PKG_VERSION ('V3.1.0 10/01/2018 MBH');
END VERSION;
--------------------------------------------------------------------------------------------------------------------
FUNCTION        PUT_ON_STG_CARD_STATISTICS_CR  (P_STG_CARD_STATIST_CR_REC  IN           STG_CARD_STATISTICS_CR%ROWTYPE)
                                                RETURN PLS_INTEGER IS
V_ENV_INFO_TRACE              GLOBAL_VARS.ENV_INFO_TRACE_TYPE:= NULL;
BEGIN
        V_ENV_INFO_TRACE.USER_NAME      :=  GLOBAL_VARS.USER_NAME;
        V_ENV_INFO_TRACE.MODULE_CODE    :=  GLOBAL_VARS.ML_STATISTICS;
        V_ENV_INFO_TRACE.PACKAGE_NAME   :=  $$PLSQL_UNIT;
        V_ENV_INFO_TRACE.FUNCTION_NAME  :=  'PUT_ON_STG_CARD_STATISTICS_CR';
        V_ENV_INFO_TRACE.LANG           :=  GLOBAL_VARS.LANG;

        INSERT INTO STG_CARD_STATISTICS_CR VALUES  P_STG_CARD_STATIST_CR_REC;
        RETURN(DECLARATION_CST.OK);
EXCEPTION
WHEN    OTHERS  THEN
        V_ENV_INFO_TRACE.ERROR_CODE   := 'PWC-00016';
        V_ENV_INFO_TRACE.PARAM1       := 'STG_CARD_STATISTICS_CR';
        V_ENV_INFO_TRACE.PARAM2       := 'STATISTICS_PERIOD = ' ||  P_STG_CARD_STATIST_CR_REC.STATISTICS_PERIOD ||
                                        ', BANK_CODE = ' || P_STG_CARD_STATIST_CR_REC.BANK_CODE ||
                                        ', NETWORK_CODE = ' || P_STG_CARD_STATIST_CR_REC.NETWORK_CODE ||
                                        ', CLASS_GROUP = ' || P_STG_CARD_STATIST_CR_REC.CLASS_GROUP ||
                                         ', CLASS = ' || P_STG_CARD_STATIST_CR_REC.CLASS ;
        V_ENV_INFO_TRACE.USER_MESSAGE :=  NULL;
        PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
        RETURN( DECLARATION_CST.ERROR );
END PUT_ON_STG_CARD_STATISTICS_CR;
--------------------------------------------------------------------------------------------------------------------
FUNCTION        PUT_ON_STG_TRANS_STATISTICS_CR  (P_STG_TRANS_STATIST_CR_REC  IN           STG_TRANSACTION_STATISTICS_CR%ROWTYPE)
                                            RETURN PLS_INTEGER IS
V_ENV_INFO_TRACE              GLOBAL_VARS.ENV_INFO_TRACE_TYPE:= NULL;
BEGIN
        V_ENV_INFO_TRACE.USER_NAME      :=  GLOBAL_VARS.USER_NAME;
        V_ENV_INFO_TRACE.MODULE_CODE    :=  GLOBAL_VARS.ML_STATISTICS;
        V_ENV_INFO_TRACE.PACKAGE_NAME   :=  $$PLSQL_UNIT;
        V_ENV_INFO_TRACE.FUNCTION_NAME  :=  'PUT_ON_STG_TRANS_STATISTICS_CR';
        V_ENV_INFO_TRACE.LANG           :=  GLOBAL_VARS.LANG;

        INSERT INTO STG_TRANSACTION_STATISTICS_CR VALUES  P_STG_TRANS_STATIST_CR_REC;
        RETURN(DECLARATION_CST.OK);
EXCEPTION
WHEN    OTHERS  THEN
        V_ENV_INFO_TRACE.ERROR_CODE   := 'PWC-00016';
        V_ENV_INFO_TRACE.PARAM1       := 'STG_TRANSACTION_STATISTICS_CR';
        V_ENV_INFO_TRACE.PARAM2       := 'STATISTICS_PERIOD = ' ||  P_STG_TRANS_STATIST_CR_REC.STATISTICS_PERIOD ||
                                        ', BANK_CODE = ' || P_STG_TRANS_STATIST_CR_REC.BANK_CODE ||
                                        ', NETWORK_CODE = ' || P_STG_TRANS_STATIST_CR_REC.NETWORK_CODE ||
                                        ', CLASS_GROUP = ' || P_STG_TRANS_STATIST_CR_REC.CLASS_GROUP ||
                                         ', CLASS = ' || P_STG_TRANS_STATIST_CR_REC.CLASS ;
        V_ENV_INFO_TRACE.USER_MESSAGE :=  NULL;
        PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
        RETURN( DECLARATION_CST.ERROR );
END PUT_ON_STG_TRANS_STATISTICS_CR;
--------------------------------------------------------------------------------------------------------------------
FUNCTION   PREP_STG_TRANSAC_STATISTICS_CR ( P_BUSINESS_DATE                     IN          DATE,
                                            P_CUTOFF_SEQUENCE                   IN          CUTOFF_FOLLOW_UP.CUTOFF_SEQUENCE%TYPE,
                                            P_LANGUAGE_1                        IN          PWC_BANK_REPORT_PARAMETERS.LANGUAGE_1%TYPE,
                                            P_LANGUAGE_2                        IN          PWC_BANK_REPORT_PARAMETERS.LANGUAGE_2%TYPE,
                                            P_LANGUAGE_3                        IN          PWC_BANK_REPORT_PARAMETERS.LANGUAGE_3%TYPE,
                                            P_BANK_RECORD                       IN          BANK%ROWTYPE,
                                            P_NETWORK_RECORD                    IN          NETWORK%ROWTYPE,
                                            P_CLASS                             IN          CARD_TYPE.CLASS%TYPE,
                                            P_CLASS_GROUP                       IN          CARD_TYPE.CLASS_GROUP%TYPE,
                                            P_START_DATE                        IN          DATE)
                                            RETURN PLS_INTEGER IS
RETURN_STATUS                           PLS_INTEGER;
V_KEY_VAL                               RESSOURCE_BUNDLE.KEY_VAL%TYPE;
V_MULTI_LANG_REPORT_V3                  MULTI_LANG_REPORT_V3;
V_BUNDLE_REPORT_V3                      BUNDLE_REPORT_V3;
V_START_DATE                            DATE;
V_COMPTEUR                              PLS_INTEGER;
V_STG_TRANS_STATISTICS_CR_REC           STG_TRANSACTION_STATISTICS_CR%ROWTYPE;
V_TRANSACTION_AMOUNT_UC                 NUMBER(18,3);
V_N_BALANCE_CATEGORY                    CR_TRANSACTION_PARAMETER.N_BALANCE_CATEGORY%TYPE;
V_CR_TRANSACTION_PARAMETER_REC          CR_TRANSACTION_PARAMETER%ROWTYPE;
V_CURRENCY_TABLE_RECORD                 CURRENCY_TABLE%ROWTYPE;

CURSOR CUR_M_STAT_TRANSACTIONS (P_BANK_CODE BANK.BANK_CODE%TYPE,P_NETWORK_CODE  NETWORK.NETWORK_CODE%TYPE , P_CLASS CARD_TYPE.CLASS%TYPE, P_CLASS_GROUP CARD_TYPE.CLASS_GROUP%TYPE, P_START_DATE    DATE) IS
    SELECT   *
    FROM     M_STAT_TRANSACTIONS
    WHERE    STATISTICS_PERIOD         = TRUNC(TO_DATE(P_START_DATE),'MONTH')
    AND      ISSUER_BANK_CODE          = P_BANK_CODE
    AND      CLASS                     = P_CLASS
    AND      CLASS_GROUP               = P_CLASS_GROUP
    AND      NETWORK_CODE              = P_NETWORK_CODE
    AND      SOURCE_TRX                = 'R' --SOURCE CR_TRANSACTION
    AND      CARD_PRODUCT_TYPE         = GLOBAL_VARS.CREDIT_CARD_TYPE;
BEGIN
      V_ENV_INFO_TRACE.USER_NAME      :=  GLOBAL_VARS.USER_NAME;
      V_ENV_INFO_TRACE.MODULE_CODE    :=  GLOBAL_VARS.ML_STATISTICS;
      V_ENV_INFO_TRACE.LANG           :=  GLOBAL_VARS.LANG;
      V_ENV_INFO_TRACE.PACKAGE_NAME   :=  $$PLSQL_UNIT;
      V_ENV_INFO_TRACE.FUNCTION_NAME  :=  'PREP_STG_TRANSAC_STATISTICS_PP';

                V_START_DATE :=  TRUNC(P_START_DATE);
                V_COMPTEUR := 1;
                LOOP
                    V_STG_TRANS_STATISTICS_CR_REC                                :=  NULL;
                    V_STG_TRANS_STATISTICS_CR_REC.BANK_CODE                      :=  P_BANK_RECORD.BANK_CODE;
                    V_STG_TRANS_STATISTICS_CR_REC.BANK_NAME                      :=  P_BANK_RECORD.BANK_NAME;
                    V_STG_TRANS_STATISTICS_CR_REC.NETWORK_CODE                   :=  P_NETWORK_RECORD.NETWORK_CODE;
                    V_STG_TRANS_STATISTICS_CR_REC.NETWORK_NAME                   :=  P_NETWORK_RECORD.NETWORK_LABEL;
                    V_STG_TRANS_STATISTICS_CR_REC.BUSINESS_DATE                  :=  P_BUSINESS_DATE;
                    V_STG_TRANS_STATISTICS_CR_REC.CUTOFF_ID                      :=  P_CUTOFF_SEQUENCE;
                    V_STG_TRANS_STATISTICS_CR_REC.LANGUAGE_1                     :=  P_LANGUAGE_1;
                    V_STG_TRANS_STATISTICS_CR_REC.LANGUAGE_2                     :=  P_LANGUAGE_2;
                    V_STG_TRANS_STATISTICS_CR_REC.LANGUAGE_3                     :=  P_LANGUAGE_3;
                    V_STG_TRANS_STATISTICS_CR_REC.STATISTICS_PERIOD              :=  V_START_DATE;
                    V_STG_TRANS_STATISTICS_CR_REC.CLASS_GROUP                    :=  P_CLASS_GROUP;
                    V_KEY_VAL := '';
                    CASE    V_STG_TRANS_STATISTICS_CR_REC.CLASS_GROUP
                    WHEN    'IN'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS_GROUP.IN';
                    WHEN    'BU'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS_GROUP.BU';
                    ELSE    V_KEY_VAL := 'NA';
                    END CASE;
                    V_BUNDLE_REPORT_V3 := PCRD_REPORTING_TOOLS_V3.NEW_BUNDLE_REPORT_V3;
                    V_BUNDLE_REPORT_V3.KEY_VAL                      := V_KEY_VAL;
                    V_BUNDLE_REPORT_V3.LANGUAGE_1                   := V_STG_TRANS_STATISTICS_CR_REC.LANGUAGE_1;
                    V_BUNDLE_REPORT_V3.LANGUAGE_2                   := V_STG_TRANS_STATISTICS_CR_REC.LANGUAGE_2;
                    V_BUNDLE_REPORT_V3.LANGUAGE_3                   := V_STG_TRANS_STATISTICS_CR_REC.LANGUAGE_3;
                    RETURN_STATUS   :=  PCRD_REPORTING_TOOLS_V3.GET_REPORT_LABELS   ( V_BUNDLE_REPORT_V3 );
                    IF RETURN_STATUS <> DECLARATION_CST.OK
                    THEN
                        V_ENV_INFO_TRACE.USER_MESSAGE   :=  'ERROR RETURNED BY PCRD_REPORTING_TOOLS_V3.GET_REPORT_LABELS'  ||
                                                            ',BUNDLE : ' || PCRD_REPORTING_TOOLS_V3.BUNDLE ||
                                                            ',KEY_VAL : ' || V_KEY_VAL;
                        PCRD_GENERAL_TOOLS.PUT_TRACES  (V_ENV_INFO_TRACE, $$PLSQL_LINE  );
                        RETURN  ( DECLARATION_CST.ERROR );
                    END IF;
                    V_STG_TRANS_STATISTICS_CR_REC.CLASS_GROUP_DESC_1             :=  V_BUNDLE_REPORT_V3.VALUE_1;
                    V_STG_TRANS_STATISTICS_CR_REC.CLASS_GROUP_DESC_2             :=  V_BUNDLE_REPORT_V3.VALUE_2;
                    V_STG_TRANS_STATISTICS_CR_REC.CLASS_GROUP_DESC_3             :=  V_BUNDLE_REPORT_V3.VALUE_3;
                    V_STG_TRANS_STATISTICS_CR_REC.CLASS                          :=  P_CLASS;
                    V_KEY_VAL := '';
                    CASE    V_STG_TRANS_STATISTICS_CR_REC.CLASS
                    WHEN    '01'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS.1';
                    WHEN    '02'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS.2';
                    WHEN    '03'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS.3';
                    WHEN    '04'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS.4';
                    WHEN    '05'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS.5';
                    WHEN    '06'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS.6';
                    ELSE    V_KEY_VAL := 'NA';
                    END CASE;
                    V_BUNDLE_REPORT_V3 := PCRD_REPORTING_TOOLS_V3.NEW_BUNDLE_REPORT_V3;
                    V_BUNDLE_REPORT_V3.KEY_VAL                      := V_KEY_VAL;
                    V_BUNDLE_REPORT_V3.LANGUAGE_1                   := V_STG_TRANS_STATISTICS_CR_REC.LANGUAGE_1;
                    V_BUNDLE_REPORT_V3.LANGUAGE_2                   := V_STG_TRANS_STATISTICS_CR_REC.LANGUAGE_2;
                    V_BUNDLE_REPORT_V3.LANGUAGE_3                   := V_STG_TRANS_STATISTICS_CR_REC.LANGUAGE_3;
                    RETURN_STATUS   :=  PCRD_REPORTING_TOOLS_V3.GET_REPORT_LABELS   ( V_BUNDLE_REPORT_V3 );
                    IF RETURN_STATUS <> DECLARATION_CST.OK
                    THEN
                        V_ENV_INFO_TRACE.USER_MESSAGE   :=  'ERROR RETURNED BY PCRD_REPORTING_TOOLS_V3.GET_REPORT_LABELS'  ||
                                                            ',BUNDLE : ' || PCRD_REPORTING_TOOLS_V3.BUNDLE ||
                                                            ',KEY_VAL : ' || V_KEY_VAL;
                        PCRD_GENERAL_TOOLS.PUT_TRACES  (V_ENV_INFO_TRACE, $$PLSQL_LINE  );
                        RETURN  ( DECLARATION_CST.ERROR );
                    END IF;
                    V_STG_TRANS_STATISTICS_CR_REC.CLASS_DESCRIPTION_1                 := V_BUNDLE_REPORT_V3.VALUE_1;
                    V_STG_TRANS_STATISTICS_CR_REC.CLASS_DESCRIPTION_2                 := V_BUNDLE_REPORT_V3.VALUE_2;
                    V_STG_TRANS_STATISTICS_CR_REC.CLASS_DESCRIPTION_3                 := V_BUNDLE_REPORT_V3.VALUE_3;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_SALES_TRX                   := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_SALES_TRX                 := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_SALES_ONUS_TRX              := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_SALES_ONUS_TRX            := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_SALES_NAT_IN_TRX            := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_SALES_NAT_IN_TRX          := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_SALES_INTL_TRX              := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_SALES_INTL_TRX            := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_SALES_EUR_TRX               := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_SALES_EUR_TRX             := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_CASH_TRX                    := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_CASH_TRX                  := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_CASH_ONUS_TRX               := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_CASH_ONUS_TRX             := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_CASH_NAT_IN_TRX             := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_CASH_NAT_IN_TRX           := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_CASH_INTL_TRX               := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_CASH_INTL_TRX             := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_CASH_EUR_TRX                := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_CASH_EUR_TRX              := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_ATM_TRX                     := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_ATM_TRX                   := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_ATM_ONUS_TRX                := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_ATM_ONUS_TRX              := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_ATM_NAT_IN_TRX              := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_ATM_NAT_IN_TRX            := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_ATM_INTL_TRX                := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_ATM_INTL_TRX              := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_ATM_EUR_TRX                 := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_ATM_EUR_TRX               := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_CR_VOUCHER_TRX              := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_CR_VOUCHER_TRX            := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_CR_VOUCHER_ONUS_TRX         := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_CR_VOUCHER_ONUS_TRX         := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_CR_VOUCHER_NAT_TRX          := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_CR_VOUCHER_NAT_TRX          := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_CR_VOUCHER_INTL_TRX         := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_CR_VOUCHER_INTL_TRX         := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_CR_VOUCHER_EUR_TRX          := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_CR_VOUCHER_EUR_TRX          := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_ORG_CR_TRX                  := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_ORG_CR_TRX                := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_ORG_CR_ONUS_TRX             := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_ORG_CR_ONUS_TRX             := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_ORG_CR_NAT_TRX              := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_ORG_CR_NAT_TRX              := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_ORG_CR_INTL_TRX             := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_ORG_CR_INTL_TRX             := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_ORG_CR_EUR_TRX              := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_ORG_CR_EUR_TRX              := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_AFT_TRX                     := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_AFT_TRX                   := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_AFT_ONUS_TRX                := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_AFT_ONUS_TRX                := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_AFT_NAT_TRX                 := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_AFT_NAT_TRX                 := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_AFT_INTL_TRX                := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_AFT_INTL_TRX                := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_AFT_EUR_TRX                 := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_AFT_EUR_TRX                 := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_ALL_TRX                     := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_ALL_TRX                   := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_ALL_ONUS_TRX                := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_ALL_ONUS_TRX              := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_ALL_NAT_IN_TRX              := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_ALL_NAT_IN_TRX            := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_ALL_INTL_TRX                := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_ALL_INTL_TRX              := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_ALL_EUR_TRX                 := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_ALL_EUR_TRX               := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_PUR_CHIP_TRX                := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_PUR_CHIP_TRX                := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_PUR_CHIP_ONUS_TRX           := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_PUR_CHIP_ONUS_TRX           := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_PUR_CHIP_NAT_IN_TRX         := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_PUR_CHIP_NAT_IN_TRX         := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_PUR_CHIP_INTL_TRX           := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_PUR_CHIP_INTL_TRX         := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_PUR_CHIP_EUR_TRX            := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_PUR_CHIP_EUR_TRX          := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_PUR_ECOM_TRX                := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_PUR_ECOM_TRX                := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_PUR_ECOM_ONUS_TRX           := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_PUR_ECOM_ONUS_TRX           := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_PUR_ECOM_NAT_IN_TRX         := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_PUR_ECOM_NAT_IN_TRX         := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_PUR_ECOM_INTL_TRX           := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_PUR_ECOM_INTL_TRX         := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_PUR_ECOM_EUR_TRX            := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_PUR_ECOM_EUR_TRX          := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_CASHBACK_TRX                := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_CASHBACK_TRX                := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_CASHBACK_ONUS_TRX           := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_CASHBACK_ONUS_TRX           := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_CASHBACK_NAT_IN_TRX         := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_CASHBACK_NAT_IN_TRX         := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_CASHBACK_INTL_TRX           := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_CASHBACK_INTL_TRX         := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_CASHBACK_EUR_TRX            := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_CASHBACK_EUR_TRX          := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_BAL_TRNS_TRX                := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_BAL_TRNS_TRX                := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_BAL_TRNS_ONUS_TRX           := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_BAL_TRNS_ONUS_TRX           := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_BAL_TRNS_NAT_IN_TRX         := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_BAL_TRNS_NAT_IN_TRX         := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_BAL_TRNS_INTL_TRX           := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_BAL_TRNS_INTL_TRX         := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_BAL_TRNS_EUR_TRX            := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_BAL_TRNS_EUR_TRX          := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.QUARTERLY_TOTALS                    := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_OUTSTAND_EOF_PRIOR_QUARTER      := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_OUTSTANDING                     := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOTAL_DEBITS                        := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOTAL_CREDITS                       := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.FINANCE_CHARGES                     := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.ACCOUNT_CHARGES_AND_FEES            := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.MISCELLANEOUS_DEBITS_ADJUST         := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.PAYMENTS_RECEIVED                   := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.CREDITS_VOUCHER                     := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.OTHER_CREDITS                       := 0.;
                    -- START HFZ20221114
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_CRD_PRS_SLS_ONUS_TRX   := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_CRD_PRS_SLS_ONUS_TRX   := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_CRD_PRS_SLS_NAT_IN_TRX := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_CRD_PRS_SLS_NAT_IN_TRX := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_CRD_PRS_SLS_INTL_TRX   := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_CRD_PRS_SLS_INTL_TRX   := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_CD_NT_PRS_SLS_ONUS_TRX := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_CD_NT_PRS_SLS_ONUS_TRX := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NB_CD_NT_PRS_SLS_NAT_IN_TX := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VL_CD_NT_PRS_SLS_NAT_IN_TX := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_CD_NT_PRS_SLS_INTL_TRX := 0.;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_CD_NT_PRS_SLS_INTL_TRX := 0.;
                    -- END   HFZ20221114
                    FOR V_M_STAT_TRANSACTIONS_REC IN CUR_M_STAT_TRANSACTIONS (P_BANK_RECORD.BANK_CODE,P_NETWORK_RECORD.NETWORK_CODE,P_CLASS,P_CLASS_GROUP,V_START_DATE)
                    LOOP
                        RETURN_STATUS   :=  PCRD_GET_PARAM_REVOLVING_ROWS.GET_CR_TRANSACTION_PARAMETER_1 (  P_BANK_RECORD.BANK_CODE,
                                                                                                            V_M_STAT_TRANSACTIONS_REC.TRANSACTION_CODE,
                                                                                                            V_CR_TRANSACTION_PARAMETER_REC);
                        IF RETURN_STATUS <> DECLARATION_CST.OK
                        THEN
                            V_ENV_INFO_TRACE.USER_MESSAGE   := 'OTHERS ERROR BANK_CODE : ' || P_BANK_RECORD.BANK_CODE
                                                              ||' TRANSACTION_CODE: ' || V_M_STAT_TRANSACTIONS_REC.TRANSACTION_CODE;
                            PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
                            RETURN DECLARATION_CST.ERROR;
                        END IF ;
                        --START MBH10012018
                        IF(V_M_STAT_TRANSACTIONS_REC.CARDHOLDER_TRANSACTION_SIGN = 'D')
                        THEN
                            V_TRANSACTION_AMOUNT_UC :=  V_M_STAT_TRANSACTIONS_REC.TRANSACTION_AMOUNT_UC;--V_M_STAT_TRANSACTIONS_REC.TRANSACTION_AMOUNT_LC;
                        ELSE
                            V_TRANSACTION_AMOUNT_UC :=  V_M_STAT_TRANSACTIONS_REC.TRANSACTION_AMOUNT_UC * -1;--V_M_STAT_TRANSACTIONS_REC.TRANSACTION_AMOUNT_LC * -1
                        END IF;
                        V_STG_TRANS_STATISTICS_CR_REC.CURRENCY_CODE        :=  V_M_STAT_TRANSACTIONS_REC.USER_CURRENCY_CODE;--V_M_STAT_TRANSACTIONS_REC.LOCAL_CURRENCY_CODE;
                        V_STG_TRANS_STATISTICS_CR_REC.CURRENCY_EXPONENT    :=  V_M_STAT_TRANSACTIONS_REC.USER_CURRENCY_EXP;--V_M_STAT_TRANSACTIONS_REC.LOCAL_CURRENCY_EXP;
                        --END MBH10012018
                        IF (V_M_STAT_TRANSACTIONS_REC.SETTLEMENT_FLAG IS NOT NULL)
                        THEN
                            IF V_M_STAT_TRANSACTIONS_REC.SETTLEMENT_FLAG  = '8'  OR V_M_STAT_TRANSACTIONS_REC.ORIGIN_CODE = GLOBAL_VARS.ORIGIN_CODE_C_ONUS_M_NATIONAL -- TED02062016
                            THEN
                                IF  (   V_M_STAT_TRANSACTIONS_REC.TRANSACTION_CODE  IN (GLOBAL_VARS.TC_PURCHASE,GLOBAL_VARS.TC_UNIQUE))
                                THEN
                                -- START HFZ20221114
                                    IF (SUBSTR(V_M_STAT_TRANSACTIONS_REC.POS_DATA, 6, 1) = '1')
                                    THEN
                                         V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_CRD_PRS_SLS_NAT_IN_TRX   := NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_CRD_PRS_SLS_NAT_IN_TRX,0)
                                                                                                            +   V_TRANSACTION_AMOUNT_UC;
                                         V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_CRD_PRS_SLS_NAT_IN_TRX   := NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_CRD_PRS_SLS_NAT_IN_TRX,0)
                                                                                                            +   V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION;
                                    ELSE
                                        V_STG_TRANS_STATISTICS_CR_REC.TOT_NB_CD_NT_PRS_SLS_NAT_IN_TX    := NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NB_CD_NT_PRS_SLS_NAT_IN_TX,0)
                                                                                                            +   V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION;
                                        V_STG_TRANS_STATISTICS_CR_REC.TOT_VL_CD_NT_PRS_SLS_NAT_IN_TX    := NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VL_CD_NT_PRS_SLS_NAT_IN_TX,0)
                                                                                                            +   V_TRANSACTION_AMOUNT_UC;
                                    END IF;--END HFZ20221114
                                    IF (NVL(V_M_STAT_TRANSACTIONS_REC.POS_ENTRY_MODE,'X') = 'C')
                                    THEN
                                            V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_PUR_CHIP_NAT_IN_TRX     :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_PUR_CHIP_NAT_IN_TRX,0)
                                                                                                        +   V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION;
                                            V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_PUR_CHIP_NAT_IN_TRX     :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_PUR_CHIP_NAT_IN_TRX,0)
                                                                                                        +   V_TRANSACTION_AMOUNT_UC;
                                    ELSIF (NVL(V_M_STAT_TRANSACTIONS_REC.POS_ENTRY_MODE,'X') = 'E')
                                    THEN
                                            V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_PUR_ECOM_NAT_IN_TRX     :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_PUR_ECOM_NAT_IN_TRX,0)
                                                                                                        +   V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION;
                                            V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_PUR_ECOM_NAT_IN_TRX     :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_PUR_ECOM_NAT_IN_TRX,0)
                                                                                                        +   V_TRANSACTION_AMOUNT_UC;
                                    ELSE
                                            V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_SALES_NAT_IN_TRX         :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_SALES_NAT_IN_TRX,0)
                                                                                                        +   V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION;
                                            V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_SALES_NAT_IN_TRX       :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_SALES_NAT_IN_TRX,0)
                                                                                                        +   V_TRANSACTION_AMOUNT_UC;
                                    END IF;
                                END IF;
                                IF   (   V_M_STAT_TRANSACTIONS_REC.MCC   =   '6010' )
                                THEN
                                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_CASH_NAT_IN_TRX          :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_CASH_NAT_IN_TRX,0)
                                                                                                    +   V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION;
                                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_CASH_NAT_IN_TRX        :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_CASH_NAT_IN_TRX,0)
                                                                                                    +   V_TRANSACTION_AMOUNT_UC;
                                END IF;
                                IF   ( V_M_STAT_TRANSACTIONS_REC.TRANSACTION_CODE = GLOBAL_VARS.TC_WITHDRAWAL)
                                THEN
                                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_ATM_NAT_IN_TRX           :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_ATM_NAT_IN_TRX,0)
                                                                                                    +   V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION;
                                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_ATM_NAT_IN_TRX         :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_ATM_NAT_IN_TRX,0)
                                                                                                    +   V_TRANSACTION_AMOUNT_UC;

                                END IF;

                                V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_ALL_NAT_IN_TRX               :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_SALES_NAT_IN_TRX,0)
                                                                                                    +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_CASH_NAT_IN_TRX,0)
                                                                                                    +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_ATM_NAT_IN_TRX,0)
                                                                                                    +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_PUR_CHIP_NAT_IN_TRX,0)
                                                                                                    +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_PUR_ECOM_NAT_IN_TRX,0);

                                V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_ALL_NAT_IN_TRX             :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_SALES_NAT_IN_TRX,0)
                                                                                                    +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_CASH_NAT_IN_TRX,0)
                                                                                                    +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_ATM_NAT_IN_TRX,0)
                                                                                                    +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_PUR_CHIP_NAT_IN_TRX,0)
                                                                                                    +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_PUR_ECOM_NAT_IN_TRX,0);

                                --ELSIF V_M_STAT_TRANSACTIONS_REC.ORIGIN_CODE    =   GLOBAL_VARS.ORIGIN_CODE_C_ONUS_M_FOREIGN
                                ELSIF V_M_STAT_TRANSACTIONS_REC.SETTLEMENT_FLAG  = '0'
                                THEN
                                    IF  (   V_M_STAT_TRANSACTIONS_REC.TRANSACTION_CODE  IN (GLOBAL_VARS.TC_PURCHASE,GLOBAL_VARS.TC_UNIQUE) )
                                    THEN
                                    -- START HFZ20221114
                                        IF (SUBSTR(V_M_STAT_TRANSACTIONS_REC.POS_DATA, 6, 1) = '1')
                                        THEN
                                             V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_CRD_PRS_SLS_INTL_TRX    :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_CRD_PRS_SLS_INTL_TRX,0)
                                                                                                            +   V_TRANSACTION_AMOUNT_UC;
                                             V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_CRD_PRS_SLS_INTL_TRX        := NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_CRD_PRS_SLS_INTL_TRX,0)
                                                                                                            +   V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION;
                                        ELSE
                                            V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_CD_NT_PRS_SLS_INTL_TRX    := NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_CD_NT_PRS_SLS_INTL_TRX,0)
                                                                                                            +   V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION;
                                            V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_CD_NT_PRS_SLS_INTL_TRX    :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_CD_NT_PRS_SLS_INTL_TRX,0)
                                                                                                            +   V_TRANSACTION_AMOUNT_UC;
                                        END IF ;--END HFZ20221114
                                        IF (NVL(V_M_STAT_TRANSACTIONS_REC.POS_ENTRY_MODE,'X') = 'C')
                                        THEN
                                            V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_PUR_CHIP_INTL_TRX     :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_PUR_CHIP_INTL_TRX,0)
                                                                                                        +   V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION;
                                            V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_PUR_CHIP_INTL_TRX     :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_PUR_CHIP_INTL_TRX,0)
                                                                                                        +   V_TRANSACTION_AMOUNT_UC;
                                        ELSIF (NVL(V_M_STAT_TRANSACTIONS_REC.POS_ENTRY_MODE,'X') = 'E')
                                        THEN
                                                V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_PUR_ECOM_INTL_TRX       :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_PUR_ECOM_INTL_TRX,0)
                                                                                                            +   V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION;
                                                V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_PUR_ECOM_INTL_TRX     :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_PUR_ECOM_INTL_TRX,0)
                                                                                                            +   V_TRANSACTION_AMOUNT_UC;
                                        ELSE
                                                V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_SALES_INTL_TRX           :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_SALES_INTL_TRX,0)
                                                                                                            +   V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION;
                                                V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_SALES_INTL_TRX         :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_SALES_INTL_TRX,0)
                                                                                                            +   V_TRANSACTION_AMOUNT_UC;
                                        END IF;
                                    END IF;
                                    IF   (   V_M_STAT_TRANSACTIONS_REC.MCC   =   '6010')
                                    THEN
                                        V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_CASH_INTL_TRX            :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_CASH_INTL_TRX,0)
                                                                                                        +   V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION;
                                        V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_CASH_INTL_TRX          := NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_CASH_INTL_TRX,0)
                                                                                                        +   V_TRANSACTION_AMOUNT_UC;
                                    END IF;
                                    IF   (V_M_STAT_TRANSACTIONS_REC.TRANSACTION_CODE = GLOBAL_VARS.TC_WITHDRAWAL)
                                    THEN
                                        V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_ATM_INTL_TRX             :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_ATM_INTL_TRX,0)
                                                                                                        +   V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION;
                                        V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_ATM_INTL_TRX           :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_ATM_INTL_TRX,0)
                                                                                                        +   V_TRANSACTION_AMOUNT_UC;
                                    END IF;
                                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_ALL_INTL_TRX                 :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_SALES_INTL_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_CASH_INTL_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_ATM_INTL_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_PUR_CHIP_INTL_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_PUR_ECOM_INTL_TRX,0);

                                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_ALL_INTL_TRX               :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_SALES_INTL_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_CASH_INTL_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_ATM_INTL_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_PUR_CHIP_INTL_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_PUR_ECOM_INTL_TRX,0);

                                END IF;
                            ELSE
                                IF V_M_STAT_TRANSACTIONS_REC.ORIGIN_CODE    =   GLOBAL_VARS.ORIGIN_CODE_C_ONUS_M_LOCAL
                                THEN
                                    IF  (   V_M_STAT_TRANSACTIONS_REC.TRANSACTION_CODE  IN (GLOBAL_VARS.TC_PURCHASE,GLOBAL_VARS.TC_UNIQUE))
                                    THEN
                                     -- START HFZ20221114
                                        IF (SUBSTR(V_M_STAT_TRANSACTIONS_REC.POS_DATA, 6, 1) = '1')
                                        THEN
                                             V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_CRD_PRS_SLS_ONUS_TRX     :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_CRD_PRS_SLS_ONUS_TRX,0)
                                                                                                            +   V_TRANSACTION_AMOUNT_UC;
                                             V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_CRD_PRS_SLS_ONUS_TRX     :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_CRD_PRS_SLS_ONUS_TRX,0)
                                                                                                            +   V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION;
                                        ELSE
                                            V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_CD_NT_PRS_SLS_ONUS_TRX    :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_CD_NT_PRS_SLS_ONUS_TRX,0)
                                                                                                            +   V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION;
                                            V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_CD_NT_PRS_SLS_ONUS_TRX    :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_CD_NT_PRS_SLS_ONUS_TRX,0)
                                                                                                            +   V_TRANSACTION_AMOUNT_UC;
                                        END IF ;--END HFZ20221114
                                        IF (NVL(V_M_STAT_TRANSACTIONS_REC.POS_ENTRY_MODE,'X') = 'C')
                                        THEN
                                            V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_PUR_CHIP_ONUS_TRX     :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_PUR_CHIP_ONUS_TRX,0)
                                                                                                        +   V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION;
                                            V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_PUR_CHIP_ONUS_TRX     :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_PUR_CHIP_ONUS_TRX,0)
                                                                                                        +   V_TRANSACTION_AMOUNT_UC;
                                        ELSIF (NVL(V_M_STAT_TRANSACTIONS_REC.POS_ENTRY_MODE,'X') = 'E')
                                        THEN
                                                V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_PUR_ECOM_ONUS_TRX       :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_PUR_ECOM_ONUS_TRX,0)
                                                                                                            +   V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION;
                                                V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_PUR_ECOM_ONUS_TRX       :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_PUR_ECOM_ONUS_TRX,0)
                                                                                                            +   V_TRANSACTION_AMOUNT_UC;
                                        ELSE
                                                V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_SALES_ONUS_TRX           :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_SALES_ONUS_TRX,0)
                                                                                                            +   NVL(V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION,0);
                                                V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_SALES_ONUS_TRX         :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_SALES_ONUS_TRX,0)
                                                                                                            +   NVL(V_TRANSACTION_AMOUNT_UC,0);
                                        END IF;
                                    END IF;
                                    IF   (   V_M_STAT_TRANSACTIONS_REC.MCC   =   '6010' )
                                    THEN
                                        V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_CASH_ONUS_TRX            :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_CASH_ONUS_TRX,0)
                                                                                                        +   NVL(V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION,0);
                                        V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_CASH_ONUS_TRX          :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_CASH_ONUS_TRX,0)
                                                                                                        +   NVL(V_TRANSACTION_AMOUNT_UC,0);
                                    END IF;
                                    IF  (V_M_STAT_TRANSACTIONS_REC.TRANSACTION_CODE = GLOBAL_VARS.TC_WITHDRAWAL)
                                    THEN
                                        V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_ATM_ONUS_TRX             :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_ATM_ONUS_TRX,0)
                                                                                                        +   NVL(V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION,0);
                                        V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_ATM_ONUS_TRX           :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_ATM_ONUS_TRX,0)
                                                                                                        +   NVL(V_TRANSACTION_AMOUNT_UC,0);
                                    END IF;
                                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_ALL_ONUS_TRX                 :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_SALES_ONUS_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_CASH_ONUS_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_ATM_ONUS_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_PUR_ECOM_ONUS_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_PUR_CHIP_ONUS_TRX,0);

                                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_ALL_ONUS_TRX               :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_SALES_ONUS_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_CASH_ONUS_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_ATM_ONUS_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_PUR_ECOM_ONUS_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_PUR_CHIP_ONUS_TRX,0);
                                ELSIF (V_M_STAT_TRANSACTIONS_REC.ORIGIN_CODE    =   GLOBAL_VARS.ORIGIN_CODE_C_ONUS_M_NATIONAL)
                                THEN
                                    IF  (   V_M_STAT_TRANSACTIONS_REC.TRANSACTION_CODE  IN (GLOBAL_VARS.TC_PURCHASE,GLOBAL_VARS.TC_UNIQUE))
                                    THEN
                                        IF (NVL(V_M_STAT_TRANSACTIONS_REC.POS_ENTRY_MODE,'X') = 'C')
                                        THEN
                                                V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_PUR_CHIP_NAT_IN_TRX     :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_PUR_CHIP_NAT_IN_TRX,0)
                                                                                                            +   V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION;
                                                V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_PUR_CHIP_NAT_IN_TRX     :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_PUR_CHIP_NAT_IN_TRX,0)
                                                                                                            +   V_TRANSACTION_AMOUNT_UC;
                                        ELSIF (NVL(V_M_STAT_TRANSACTIONS_REC.POS_ENTRY_MODE,'X') = 'E')
                                        THEN
                                                V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_PUR_ECOM_NAT_IN_TRX     :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_PUR_ECOM_NAT_IN_TRX,0)
                                                                                                            +   V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION;
                                                V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_PUR_ECOM_NAT_IN_TRX     :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_PUR_ECOM_NAT_IN_TRX,0)
                                                                                                            +   V_TRANSACTION_AMOUNT_UC;
                                        ELSE
                                                V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_SALES_NAT_IN_TRX         :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_SALES_NAT_IN_TRX,0)
                                                                                                            +   V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION;
                                                V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_SALES_NAT_IN_TRX       :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_SALES_NAT_IN_TRX,0)
                                                                                                            +   V_TRANSACTION_AMOUNT_UC;
                                        END IF;
                                    END IF;
                                    IF   (   V_M_STAT_TRANSACTIONS_REC.MCC   =   '6010' )
                                    THEN
                                        V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_CASH_NAT_IN_TRX          :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_CASH_NAT_IN_TRX,0)
                                                                                                        +   V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION;
                                        V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_CASH_NAT_IN_TRX        :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_CASH_NAT_IN_TRX,0)
                                                                                                        +   V_TRANSACTION_AMOUNT_UC;
                                    END IF;
                                    IF (V_M_STAT_TRANSACTIONS_REC.TRANSACTION_CODE = GLOBAL_VARS.TC_WITHDRAWAL)
                                    THEN
                                        V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_ATM_NAT_IN_TRX           :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_ATM_NAT_IN_TRX,0)
                                                                                                        +   V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION;
                                        V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_ATM_NAT_IN_TRX         :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_ATM_NAT_IN_TRX,0)
                                                                                                        +   V_TRANSACTION_AMOUNT_UC;

                                    END IF;

                                    V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_ALL_NAT_IN_TRX               :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_SALES_NAT_IN_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_CASH_NAT_IN_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_ATM_NAT_IN_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_PUR_CHIP_NAT_IN_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_PUR_ECOM_NAT_IN_TRX,0);

                                    V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_ALL_NAT_IN_TRX             :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_SALES_NAT_IN_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_CASH_NAT_IN_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_ATM_NAT_IN_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_PUR_CHIP_NAT_IN_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_PUR_ECOM_NAT_IN_TRX,0);
                                    END IF;
                            END IF;
                            V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_SALES_TRX        :=     NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_SALES_ONUS_TRX,0)
                                                                                            +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_SALES_INTL_TRX,0)
                                                                                            +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_SALES_NAT_IN_TRX,0);

                            V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_SALES_TRX      :=      NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_SALES_ONUS_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_SALES_INTL_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_SALES_NAT_IN_TRX,0);

                            V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_PUR_CHIP_TRX     :=     NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_PUR_CHIP_ONUS_TRX,0)
                                                                                            +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_PUR_CHIP_NAT_IN_TRX,0)
                                                                                            +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_PUR_CHIP_INTL_TRX,0);

                            V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_PUR_CHIP_TRX     :=      NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_PUR_CHIP_ONUS_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_PUR_CHIP_NAT_IN_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_PUR_CHIP_INTL_TRX,0);

                            V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_PUR_ECOM_TRX     :=     NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_PUR_ECOM_ONUS_TRX,0)
                                                                                            +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_PUR_ECOM_NAT_IN_TRX,0)
                                                                                            +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_PUR_ECOM_INTL_TRX,0);

                            V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_PUR_ECOM_TRX     :=      NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_PUR_ECOM_ONUS_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VAL_PUR_ECOM_NAT_IN_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_PUR_ECOM_INTL_TRX,0);

                            V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_CASH_TRX        :=     NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_CASH_ONUS_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_CASH_INTL_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_CASH_NAT_IN_TRX,0);

                            V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_CASH_TRX      :=      NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_CASH_ONUS_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_CASH_INTL_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_CASH_NAT_IN_TRX,0);

                            V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_ATM_TRX          :=     NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_ATM_ONUS_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_ATM_INTL_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_ATM_NAT_IN_TRX,0);

                            V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_ATM_TRX        :=      NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_ATM_ONUS_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_ATM_INTL_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_ATM_NAT_IN_TRX,0);

                            V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_ALL_TRX          :=      NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_ALL_ONUS_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_ALL_NAT_IN_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_NBR_ALL_INTL_TRX,0);

                            V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_ALL_TRX        :=      NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_ALL_ONUS_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_ALL_NAT_IN_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_ALL_INTL_TRX,0);


                            IF  ((V_M_STAT_TRANSACTIONS_REC.REVERSAL_FLAG =  'N' AND V_CR_TRANSACTION_PARAMETER_REC.N_BALANCE_CATEGORY = 6)
                                OR (V_M_STAT_TRANSACTIONS_REC.REVERSAL_FLAG =  'R' AND V_CR_TRANSACTION_PARAMETER_REC.R_BALANCE_CATEGORY = 6))
                            THEN
                                V_STG_TRANS_STATISTICS_CR_REC.FINANCE_CHARGES                  :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.FINANCE_CHARGES,0)
                                                                                                +   V_TRANSACTION_AMOUNT_UC;
                            END IF;
                            IF  ((V_M_STAT_TRANSACTIONS_REC.REVERSAL_FLAG =  'N' AND (V_CR_TRANSACTION_PARAMETER_REC.N_BALANCE_CATEGORY = 8 OR V_CR_TRANSACTION_PARAMETER_REC.N_BALANCE_CATEGORY = 9)) --SAB061020115
                                OR (V_M_STAT_TRANSACTIONS_REC.REVERSAL_FLAG =  'R' AND (V_CR_TRANSACTION_PARAMETER_REC.R_BALANCE_CATEGORY = 8 OR V_CR_TRANSACTION_PARAMETER_REC.R_BALANCE_CATEGORY = 9))) --SAB061020115
                            THEN
                                V_STG_TRANS_STATISTICS_CR_REC.ACCOUNT_CHARGES_AND_FEES :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.ACCOUNT_CHARGES_AND_FEES,0)
                                                                                             +   V_TRANSACTION_AMOUNT_UC;
                             END IF;
                            IF  ((V_M_STAT_TRANSACTIONS_REC.REVERSAL_FLAG =  'N' AND V_CR_TRANSACTION_PARAMETER_REC.N_BALANCE_CATEGORY = 11)
                                OR (V_M_STAT_TRANSACTIONS_REC.REVERSAL_FLAG =  'R' AND V_CR_TRANSACTION_PARAMETER_REC.R_BALANCE_CATEGORY = 11))
                            THEN
                                V_STG_TRANS_STATISTICS_CR_REC.PAYMENTS_RECEIVED                :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.PAYMENTS_RECEIVED,0)
                                                                                                +   V_TRANSACTION_AMOUNT_UC;
                             END IF;
                             IF  V_M_STAT_TRANSACTIONS_REC.TRANSACTION_CODE   IN (GLOBAL_VARS.TC_ACCOUNT_MISC_DEBIT,
                                                                                     GLOBAL_VARS.TC_CARD_MISC_DEBIT,
                                                                                     GLOBAL_VARS.TC_CARD_MISC_DEBIT_1,
                                                                                     GLOBAL_VARS.TC_CARD_MISC_DEBIT_2,
                                                                                     GLOBAL_VARS.TC_CARD_MISC_DEBIT_3,
                                                                                     GLOBAL_VARS.TC_CARD_MISC_DEBIT_4 )
                            THEN
                                V_STG_TRANS_STATISTICS_CR_REC.MISCELLANEOUS_DEBITS_ADJUST             :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.MISCELLANEOUS_DEBITS_ADJUST,0)
                                                                                                            +   V_TRANSACTION_AMOUNT_UC;
                            END IF;
                            IF  V_M_STAT_TRANSACTIONS_REC.TRANSACTION_CODE = GLOBAL_VARS.TC_CREDIT_VOUCHER
                            THEN
                                V_STG_TRANS_STATISTICS_CR_REC.CREDITS_VOUCHER                  :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.CREDITS_VOUCHER,0)
                                                                                                    +   V_TRANSACTION_AMOUNT_UC;
                            END IF;
                            IF  V_M_STAT_TRANSACTIONS_REC.TRANSACTION_CODE   IN (GLOBAL_VARS.TC_ACCOUNT_MISC_CREDIT,
                                                                                    GLOBAL_VARS.TC_CARD_MISC_CREDIT,
                                                                                    GLOBAL_VARS.TC_CARD_MISC_CREDIT_1)
                            THEN
                                V_STG_TRANS_STATISTICS_CR_REC.OTHER_CREDITS                    :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.OTHER_CREDITS,0)
                                                                                                    +   V_TRANSACTION_AMOUNT_UC;
                            END IF;

                            V_STG_TRANS_STATISTICS_CR_REC.TOTAL_DEBITS                         :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_VALUE_ALL_TRX,0)
                                                                                                +   NVL(V_STG_TRANS_STATISTICS_CR_REC.FINANCE_CHARGES,0)
                                                                                                +   NVL(V_STG_TRANS_STATISTICS_CR_REC.ACCOUNT_CHARGES_AND_FEES,0)
                                                                                                +   NVL(V_STG_TRANS_STATISTICS_CR_REC.MISCELLANEOUS_DEBITS_ADJUST,0);

                            V_STG_TRANS_STATISTICS_CR_REC.TOTAL_CREDITS                        :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.PAYMENTS_RECEIVED,0)
                                                                                                +   NVL(V_STG_TRANS_STATISTICS_CR_REC.CREDITS_VOUCHER,0)
                                                                                                +   NVL(V_STG_TRANS_STATISTICS_CR_REC.OTHER_CREDITS,0);

                    END LOOP;
                    IF (V_STG_TRANS_STATISTICS_CR_REC.CURRENCY_CODE IS NULL)
                    THEN
                        RETURN_STATUS   :=  PCRD_GET_PARAM_GENERAL_ROWS.GET_CURRENCY_TABLE (P_BANK_RECORD.BANK_CURRENCY_CODE,
                                                                                            V_CURRENCY_TABLE_RECORD,
                                                                                            TRUE);
                        IF RETURN_STATUS <> DECLARATION_CST.OK
                        THEN
                                V_ENV_INFO_TRACE.USER_MESSAGE := ':ERROR RETURNED BY PCRD_GET_PARAM_GENERAL_ROWS.GET_CURRENCY_TABLE';
                                PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
                                RETURN(RETURN_STATUS);
                        END IF;
                        V_STG_TRANS_STATISTICS_CR_REC.CURRENCY_CODE     :=  V_CURRENCY_TABLE_RECORD.CURRENCY_CODE;
                        V_STG_TRANS_STATISTICS_CR_REC.CURRENCY_EXPONENT :=  V_CURRENCY_TABLE_RECORD.CURRENCY_EXPONENT;
                    END IF;
                    BEGIN
                        SELECT NVL(TOT_OUTSTANDING,0)
                        INTO   V_STG_TRANS_STATISTICS_CR_REC.TOT_OUTSTAND_EOF_PRIOR_QUARTER
                        FROM   STG_TRANSACTION_STATISTICS_CR
                        WHERE       STATISTICS_PERIOD    = ADD_MONTHS(V_STG_TRANS_STATISTICS_CR_REC.STATISTICS_PERIOD,-3)
                               AND  NETWORK_CODE         = V_STG_TRANS_STATISTICS_CR_REC.NETWORK_CODE
                               AND  CLASS_GROUP          = V_STG_TRANS_STATISTICS_CR_REC.CLASS_GROUP
                               AND  CLASS                = V_STG_TRANS_STATISTICS_CR_REC.CLASS
                               AND  BANK_CODE            = V_STG_TRANS_STATISTICS_CR_REC.BANK_CODE;
                        EXCEPTION WHEN NO_DATA_FOUND
                        THEN
                            V_STG_TRANS_STATISTICS_CR_REC.TOT_OUTSTAND_EOF_PRIOR_QUARTER := 0.0;
                        WHEN OTHERS
                        THEN
                            V_ENV_INFO_TRACE.USER_MESSAGE   := 'OTHERS ERROR READ TABLE STG_TRANSACTION_STATISTICS_CR';
                            PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
                            RETURN DECLARATION_CST.ERROR;
                    END;
                    V_STG_TRANS_STATISTICS_CR_REC.TOT_OUTSTANDING           :=  NVL(V_STG_TRANS_STATISTICS_CR_REC.TOT_OUTSTAND_EOF_PRIOR_QUARTER,0)
                                                                            +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOTAL_DEBITS,0)
                                                                            +   NVL(V_STG_TRANS_STATISTICS_CR_REC.TOTAL_CREDITS,0);

                    RETURN_STATUS   := PCRD_STA_REPORTING_TOOLS_1_V3.PUT_ON_STG_TRANS_STATISTICS_CR(V_STG_TRANS_STATISTICS_CR_REC);
                    IF (RETURN_STATUS <> DECLARATION_CST.OK)
                    THEN
                        V_ENV_INFO_TRACE.USER_MESSAGE := 'ERROR RETURNED BY PCRD_STA_REPORTING_TOOLS_1_V3.PUT_ON_STG_TRANS_STATISTICS_CR STATISTICS_PERIOD = ' ||  V_STG_TRANS_STATISTICS_CR_REC.STATISTICS_PERIOD ||
                                                    ', BANK_CODE = ' || V_STG_TRANS_STATISTICS_CR_REC.BANK_CODE ||
                                                    ', NETWORK_CODE = ' || V_STG_TRANS_STATISTICS_CR_REC.NETWORK_CODE ||
                                                    ', CLASS_GROUP = ' || V_STG_TRANS_STATISTICS_CR_REC.CLASS_GROUP ||
                                                     ', CLASS = ' || V_STG_TRANS_STATISTICS_CR_REC.CLASS ;
                        PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
                        RETURN (DECLARATION_CST.ERROR);
                    END IF;
                    EXIT WHEN V_COMPTEUR = 3;
                    V_COMPTEUR := V_COMPTEUR + 1;
                    V_START_DATE    :=  ADD_MONTHS(V_START_DATE,1);
                END LOOP;
    RETURN(DECLARATION_CST.OK);
EXCEPTION WHEN OTHERS
THEN
    V_ENV_INFO_TRACE.USER_MESSAGE   :=  NULL;
    PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
    RETURN(DECLARATION_CST.NOK);
END PREP_STG_TRANSAC_STATISTICS_CR;
----------------------------------------------------------------------------------------------------------------------
FUNCTION   PREP_STG_CARD_STATISTICS_CR   (  P_BUSINESS_DATE                     IN          DATE,
                                            P_CUTOFF_SEQUENCE                   IN          CUTOFF_FOLLOW_UP.CUTOFF_SEQUENCE%TYPE,
                                            P_LANGUAGE_1                        IN          PWC_BANK_REPORT_PARAMETERS.LANGUAGE_1%TYPE,
                                            P_LANGUAGE_2                        IN          PWC_BANK_REPORT_PARAMETERS.LANGUAGE_2%TYPE,
                                            P_LANGUAGE_3                        IN          PWC_BANK_REPORT_PARAMETERS.LANGUAGE_3%TYPE,
                                            P_BANK_RECORD                       IN          BANK%ROWTYPE,
                                            P_NETWORK_RECORD                    IN          NETWORK%ROWTYPE,
                                            P_STAT_CURRENCY_TABLE_RECORD        IN          CURRENCY_TABLE%ROWTYPE,    --MBH10012018
                                            P_CLASS                             IN          CARD_TYPE.CLASS%TYPE,
                                            P_CLASS_GROUP                       IN          CARD_TYPE.CLASS_GROUP%TYPE,
                                            P_START_DATE                        IN          DATE,
                                            P_END_DATE                          IN          DATE)
                                            RETURN PLS_INTEGER IS

RETURN_STATUS                           PLS_INTEGER;
V_START_DATE                            DATE;
V_COMPTEUR                              PLS_INTEGER;
V_STG_TRANS_CARD_CR_REC                 STG_CARD_STATISTICS_CR%ROWTYPE;
V_KEY_VAL                               RESSOURCE_BUNDLE.KEY_VAL%TYPE;
V_MULTI_LANG_REPORT_V3                  MULTI_LANG_REPORT_V3;
V_BUNDLE_REPORT_V3                      BUNDLE_REPORT_V3;
V_FEES_AMOUNT                           CARD_FEES.FEES_AMOUNT_FIRST%TYPE;
--V_FEES_AMOUNT_LC                        CARD_FEES.FEES_AMOUNT_FIRST%TYPE;
V_FEES_AMOUNT_UC                        CARD_FEES.FEES_AMOUNT_FIRST%TYPE;--MBH10012018
V_RATE_ISO_STR_FORMAT                   TRANS_BASE_TRANSACTION.CONV_RATE_SETTLEMENT%TYPE;
V_RATE_DATE                             DATE;
V_NBR_RECORD                            PLS_INTEGER:=0;
V_CONVERSION_DATE                       CONVERSION_RATE.RATE_DATE%TYPE;
V_MARKUP_AMOUNT                         TRANSACTION_HIST.CONVERSION_FEES%TYPE;
V_CURRENCY_TABLE_RECORD                 CURRENCY_TABLE%ROWTYPE;

CURSOR CUR_M_STAT_CARD (P_BANK_CODE BANK.BANK_CODE%TYPE,P_NETWORK_CODE  NETWORK.NETWORK_CODE%TYPE, P_CLASS CARD_TYPE.CLASS%TYPE, P_CLASS_GROUP CARD_TYPE.CLASS_GROUP%TYPE, P_START_DATE    DATE) IS
    SELECT   *
    FROM     M_STAT_CARD
    WHERE    STATISTICS_PERIOD         = TRUNC(TO_DATE(P_START_DATE),'MONTH')
    AND      BANK_CODE                 = P_BANK_CODE
    AND      CLASS                     = P_CLASS
    AND      CLASS_GROUP               = P_CLASS_GROUP
    AND      NETWORK_CODE              = P_NETWORK_CODE
    AND      CARD_PRODUCT_TYPE         = GLOBAL_VARS.CREDIT_CARD_TYPE;

CURSOR CUR_CARD_FEES_CREDIT (P_CLASS CARD_TYPE.CLASS%TYPE, P_CLASS_GROUP CARD_TYPE.CLASS_GROUP%TYPE)
IS
SELECT A.*
FROM CARD_FEES A, CARD_PRODUCT B, CARD_TYPE C
WHERE CARD_FEES_CODE = CARD_FEES_IND_PRIM
AND A.BANK_CODE = B.BANK_CODE
AND A.BANK_CODE = C.BANK_CODE
AND B.BANK_CODE = P_BANK_RECORD.BANK_CODE
AND B.NETWORK_CARD_TYPE = C.NETWORK_CARD_TYPE
AND C.CLASS             = P_CLASS
AND C.CLASS_GROUP       = P_CLASS_GROUP
AND B.NETWORK_CODE = P_NETWORK_RECORD.NETWORK_CODE
AND B.PRODUCT_TYPE = GLOBAL_VARS.CREDIT_CARD_TYPE
AND A.STATUS <> 'U';

BEGIN
      V_ENV_INFO_TRACE.USER_NAME      :=  GLOBAL_VARS.USER_NAME;
      V_ENV_INFO_TRACE.MODULE_CODE    :=  GLOBAL_VARS.ML_STATISTICS;
      V_ENV_INFO_TRACE.LANG           :=  GLOBAL_VARS.LANG;
      V_ENV_INFO_TRACE.PACKAGE_NAME   :=  $$PLSQL_UNIT;
      V_ENV_INFO_TRACE.FUNCTION_NAME  :=  'PREP_STG_CARD_STATISTICS_CR';

    V_START_DATE :=  TRUNC(P_START_DATE);
    V_COMPTEUR := 1;
    LOOP
        V_STG_TRANS_CARD_CR_REC                                :=  NULL;
        V_STG_TRANS_CARD_CR_REC.BANK_CODE                      :=  P_BANK_RECORD.BANK_CODE;
        V_STG_TRANS_CARD_CR_REC.BANK_NAME                      :=  P_BANK_RECORD.BANK_NAME;
        V_STG_TRANS_CARD_CR_REC.NETWORK_CODE                   :=  P_NETWORK_RECORD.NETWORK_CODE;
        V_STG_TRANS_CARD_CR_REC.NETWORK_NAME                   :=  P_NETWORK_RECORD.NETWORK_LABEL;
        V_STG_TRANS_CARD_CR_REC.BUSINESS_DATE                  :=  P_BUSINESS_DATE;
        V_STG_TRANS_CARD_CR_REC.CUTOFF_ID                      :=  P_CUTOFF_SEQUENCE;
        V_STG_TRANS_CARD_CR_REC.LANGUAGE_1                     :=  P_LANGUAGE_1;
        V_STG_TRANS_CARD_CR_REC.LANGUAGE_2                     :=  P_LANGUAGE_2;
        V_STG_TRANS_CARD_CR_REC.LANGUAGE_3                     :=  P_LANGUAGE_3;
        V_STG_TRANS_CARD_CR_REC.STATISTICS_PERIOD              :=  V_START_DATE;
        V_STG_TRANS_CARD_CR_REC.CLASS_GROUP                    :=  P_CLASS_GROUP;
        V_KEY_VAL := '';
        CASE    V_STG_TRANS_CARD_CR_REC.CLASS_GROUP
        WHEN    'IN'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS_GROUP.IN';
        WHEN    'BU'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS_GROUP.BU';
        ELSE    V_KEY_VAL := 'NA';
        END CASE;
        V_BUNDLE_REPORT_V3 := PCRD_REPORTING_TOOLS_V3.NEW_BUNDLE_REPORT_V3;
        V_BUNDLE_REPORT_V3.KEY_VAL                      := V_KEY_VAL;
        V_BUNDLE_REPORT_V3.LANGUAGE_1                   := V_STG_TRANS_CARD_CR_REC.LANGUAGE_1;
        V_BUNDLE_REPORT_V3.LANGUAGE_2                   := V_STG_TRANS_CARD_CR_REC.LANGUAGE_2;
        V_BUNDLE_REPORT_V3.LANGUAGE_3                   := V_STG_TRANS_CARD_CR_REC.LANGUAGE_3;
        RETURN_STATUS   :=  PCRD_REPORTING_TOOLS_V3.GET_REPORT_LABELS   ( V_BUNDLE_REPORT_V3 );
        IF RETURN_STATUS <> DECLARATION_CST.OK
        THEN
            V_ENV_INFO_TRACE.USER_MESSAGE   :=  'ERROR RETURNED BY PCRD_REPORTING_TOOLS_V3.GET_REPORT_LABELS'  ||
                                                ',BUNDLE : ' || PCRD_REPORTING_TOOLS_V3.BUNDLE ||
                                                ',KEY_VAL : ' || V_KEY_VAL;
            PCRD_GENERAL_TOOLS.PUT_TRACES  (V_ENV_INFO_TRACE, $$PLSQL_LINE  );
            RETURN  ( DECLARATION_CST.ERROR );
        END IF;
        V_STG_TRANS_CARD_CR_REC.CLASS_GROUP_DESC_1             :=  V_BUNDLE_REPORT_V3.VALUE_1;
        V_STG_TRANS_CARD_CR_REC.CLASS_GROUP_DESC_2             :=  V_BUNDLE_REPORT_V3.VALUE_2;
        V_STG_TRANS_CARD_CR_REC.CLASS_GROUP_DESC_3             :=  V_BUNDLE_REPORT_V3.VALUE_3;
        V_STG_TRANS_CARD_CR_REC.CLASS                          :=  P_CLASS;
        V_KEY_VAL := '';
        CASE    V_STG_TRANS_CARD_CR_REC.CLASS
        WHEN    '01'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS.1';
        WHEN    '02'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS.2';
        WHEN    '03'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS.3';
        WHEN    '04'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS.4';
        WHEN    '05'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS.5';
        WHEN    '06'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS.6';
        ELSE    V_KEY_VAL := 'NA';
        END CASE;
        V_BUNDLE_REPORT_V3 := PCRD_REPORTING_TOOLS_V3.NEW_BUNDLE_REPORT_V3;
        V_BUNDLE_REPORT_V3.KEY_VAL                      := V_KEY_VAL;
        V_BUNDLE_REPORT_V3.LANGUAGE_1                   := V_STG_TRANS_CARD_CR_REC.LANGUAGE_1;
        V_BUNDLE_REPORT_V3.LANGUAGE_2                   := V_STG_TRANS_CARD_CR_REC.LANGUAGE_2;
        V_BUNDLE_REPORT_V3.LANGUAGE_3                   := V_STG_TRANS_CARD_CR_REC.LANGUAGE_3;
        RETURN_STATUS   :=  PCRD_REPORTING_TOOLS_V3.GET_REPORT_LABELS   ( V_BUNDLE_REPORT_V3 );
        IF RETURN_STATUS <> DECLARATION_CST.OK
        THEN
            V_ENV_INFO_TRACE.USER_MESSAGE   :=  'ERROR RETURNED BY PCRD_REPORTING_TOOLS_V3.GET_REPORT_LABELS'  ||
                                                ',BUNDLE : ' || PCRD_REPORTING_TOOLS_V3.BUNDLE ||
                                                ',KEY_VAL : ' || V_KEY_VAL;
            PCRD_GENERAL_TOOLS.PUT_TRACES  (V_ENV_INFO_TRACE, $$PLSQL_LINE  );
            RETURN  ( DECLARATION_CST.ERROR );
        END IF;
        V_STG_TRANS_CARD_CR_REC.CLASS_DESCRIPTION_1            :=  V_BUNDLE_REPORT_V3.VALUE_1;
        V_STG_TRANS_CARD_CR_REC.CLASS_DESCRIPTION_2            :=  V_BUNDLE_REPORT_V3.VALUE_2;
        V_STG_TRANS_CARD_CR_REC.CLASS_DESCRIPTION_3            :=  V_BUNDLE_REPORT_V3.VALUE_3;
        V_STG_TRANS_CARD_CR_REC.NB_OF_CARDS                    :=   0.;
        V_STG_TRANS_CARD_CR_REC.NB_PRIMARY_CARDS               :=   0.;
        V_STG_TRANS_CARD_CR_REC.NB_SECONDARY_CARDS             :=   0.;
        V_STG_TRANS_CARD_CR_REC.NB_NEW_CARDS                   :=   0.;
        V_STG_TRANS_CARD_CR_REC.NB_RENEWAL_CARDS               :=   0.;
        V_STG_TRANS_CARD_CR_REC.NB_CANCELLED_CARDS             :=   0.;
        V_STG_TRANS_CARD_CR_REC.NB_INACTIVE_CARDS              :=   0.;
        V_STG_TRANS_CARD_CR_REC.NB_STOPPED_CARDS               :=   0.;
        V_STG_TRANS_CARD_CR_REC.NB_OF_DOUBLE_CARDS             :=   0.;
        V_STG_TRANS_CARD_CR_REC.NB_OF_CARD_REWARD              :=   0.;
        V_STG_TRANS_CARD_CR_REC.NB_OF_PINNED_CARD              :=   0.;
        V_STG_TRANS_CARD_CR_REC.NB_OF_CARD_CHIP                :=   0.;
        V_STG_TRANS_CARD_CR_REC.NB_OF_CRD_WITH_POS_ACTIVITY    :=   0.;
        V_STG_TRANS_CARD_CR_REC.NB_OF_CRD_WITH_ATM_ACTIVITY    :=   0.;
        V_STG_TRANS_CARD_CR_REC.NB_OF_CRD_WITH_ECOM_ACTIV      :=   0.;
        V_STG_TRANS_CARD_CR_REC.NB_OF_ACCOUNTS                 :=   0.;
        V_STG_TRANS_CARD_CR_REC.NB_OF_ACTIVE_ACCOUNTS          :=   0.;
        V_STG_TRANS_CARD_CR_REC.NB_OF_RESTRICTED_ACCOUNT       :=   0.;
        V_STG_TRANS_CARD_CR_REC.NB_OF_INTERNATIONAL_ACCOUNT    :=   0.;
        V_STG_TRANS_CARD_CR_REC.NB_OF_MAILED_STATEMENTS        :=   0.;
        V_STG_TRANS_CARD_CR_REC.NB_OF_MAILED_STAT_FINAN_CHARG  :=   0.;
        V_STG_TRANS_CARD_CR_REC.ANUAL_FEES_LC                  :=   0.;
        V_STG_TRANS_CARD_CR_REC.ANUAL_FEES_UC                  :=   0.;

        FOR V_M_STAT_CARD_REC IN CUR_M_STAT_CARD (P_BANK_RECORD.BANK_CODE,P_NETWORK_RECORD.NETWORK_CODE,P_CLASS,P_CLASS_GROUP,V_START_DATE)
        LOOP
           V_STG_TRANS_CARD_CR_REC.LOCAL_CURRENCY_CODE         :=  V_M_STAT_CARD_REC.LOCAL_CURRENCY_CODE;
           V_STG_TRANS_CARD_CR_REC.LOCAL_CURRENCY_EXPONENT     :=  V_M_STAT_CARD_REC.LOCAL_CURRENCY_EXPONENT;
           V_STG_TRANS_CARD_CR_REC.USER_CURRENCY_CODE          :=  V_M_STAT_CARD_REC.USER_CURRENCY_CODE;
           V_STG_TRANS_CARD_CR_REC.USER_CURRENCY_EXPONENT      :=  V_M_STAT_CARD_REC.USER_CURRENCY_EXPONENT;
           V_STG_TRANS_CARD_CR_REC.NB_OF_CARDS                 :=  V_STG_TRANS_CARD_CR_REC.NB_OF_CARDS + V_M_STAT_CARD_REC.NB_PRIMARY_CARDS + V_M_STAT_CARD_REC.NB_SECONDARY_CARDS;
           V_STG_TRANS_CARD_CR_REC.NB_PRIMARY_CARDS            :=  V_STG_TRANS_CARD_CR_REC.NB_PRIMARY_CARDS + V_M_STAT_CARD_REC.NB_PRIMARY_CARDS;
           V_STG_TRANS_CARD_CR_REC.NB_SECONDARY_CARDS          :=  V_STG_TRANS_CARD_CR_REC.NB_SECONDARY_CARDS + V_M_STAT_CARD_REC.NB_SECONDARY_CARDS;
           V_STG_TRANS_CARD_CR_REC.NB_NEW_CARDS                :=  V_STG_TRANS_CARD_CR_REC.NB_NEW_CARDS  + V_M_STAT_CARD_REC.NB_NEW_CARDS   ;
           V_STG_TRANS_CARD_CR_REC.NB_RENEWAL_CARDS            :=  V_STG_TRANS_CARD_CR_REC.NB_RENEWAL_CARDS + V_M_STAT_CARD_REC.NB_RENEWAL_CARDS;
           V_STG_TRANS_CARD_CR_REC.NB_CANCELLED_CARDS          :=  V_STG_TRANS_CARD_CR_REC.NB_CANCELLED_CARDS + V_M_STAT_CARD_REC.NB_CANCELLED_CARDS;
           V_STG_TRANS_CARD_CR_REC.NB_INACTIVE_CARDS           :=  V_STG_TRANS_CARD_CR_REC.NB_INACTIVE_CARDS + V_M_STAT_CARD_REC.NB_INACTIVE_CARDS;
           V_STG_TRANS_CARD_CR_REC.NB_STOPPED_CARDS            :=  V_STG_TRANS_CARD_CR_REC.NB_STOPPED_CARDS + V_M_STAT_CARD_REC.NB_STOPPED_CARDS;
           V_STG_TRANS_CARD_CR_REC.NB_OF_DOUBLE_CARDS          :=  V_STG_TRANS_CARD_CR_REC.NB_OF_DOUBLE_CARDS + V_M_STAT_CARD_REC.NB_OF_DOUBLE_CARDS;
           V_STG_TRANS_CARD_CR_REC.NB_OF_CARD_REWARD           :=  V_STG_TRANS_CARD_CR_REC.NB_OF_CARD_REWARD + V_M_STAT_CARD_REC.NB_OF_CARD_REWARD;
           V_STG_TRANS_CARD_CR_REC.NB_OF_PINNED_CARD           :=  V_STG_TRANS_CARD_CR_REC.NB_OF_PINNED_CARD + V_M_STAT_CARD_REC.NB_OF_PINNED_CARD;
           V_STG_TRANS_CARD_CR_REC.NB_OF_CARD_CHIP             :=  V_STG_TRANS_CARD_CR_REC.NB_OF_CARD_CHIP + V_M_STAT_CARD_REC.NB_OF_CARD_CHIP;
           V_STG_TRANS_CARD_CR_REC.NB_OF_CRD_WITH_POS_ACTIVITY :=  V_STG_TRANS_CARD_CR_REC.NB_OF_CRD_WITH_POS_ACTIVITY + V_M_STAT_CARD_REC.NB_OF_CRD_WITH_POS_ACTIVITY;
           V_STG_TRANS_CARD_CR_REC.NB_OF_CRD_WITH_ATM_ACTIVITY :=  V_STG_TRANS_CARD_CR_REC.NB_OF_CRD_WITH_ATM_ACTIVITY + V_M_STAT_CARD_REC.NB_OF_CRD_WITH_ATM_ACTIVITY;
           V_STG_TRANS_CARD_CR_REC.NB_OF_CRD_WITH_ECOM_ACTIV   :=  V_STG_TRANS_CARD_CR_REC.NB_OF_CRD_WITH_ECOM_ACTIV + V_M_STAT_CARD_REC.NB_OF_CRD_WITH_ECOM_ACTIV;
           V_STG_TRANS_CARD_CR_REC.NB_OF_ACCOUNTS              :=  V_STG_TRANS_CARD_CR_REC.NB_OF_ACCOUNTS + V_M_STAT_CARD_REC.NB_OF_ACCOUNTS;
           V_STG_TRANS_CARD_CR_REC.NB_OF_ACTIVE_ACCOUNTS       :=  V_STG_TRANS_CARD_CR_REC.NB_OF_ACTIVE_ACCOUNTS + V_M_STAT_CARD_REC.NB_OF_ACTIVE_ACCOUNTS;
        END LOOP;
        IF (V_STG_TRANS_CARD_CR_REC.LOCAL_CURRENCY_CODE IS NULL)
        THEN
            RETURN_STATUS   :=  PCRD_GET_PARAM_GENERAL_ROWS.GET_CURRENCY_TABLE (P_BANK_RECORD.BANK_CURRENCY_CODE,
                                                                                V_CURRENCY_TABLE_RECORD,
                                                                                TRUE);
            IF RETURN_STATUS <> DECLARATION_CST.OK
            THEN
                    V_ENV_INFO_TRACE.USER_MESSAGE := ':ERROR RETURNED BY PCRD_GET_PARAM_GENERAL_ROWS.GET_CURRENCY_TABLE';
                    PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
                    RETURN(RETURN_STATUS);
            END IF;
            V_STG_TRANS_CARD_CR_REC.LOCAL_CURRENCY_CODE     :=  V_CURRENCY_TABLE_RECORD.CURRENCY_CODE;
            V_STG_TRANS_CARD_CR_REC.LOCAL_CURRENCY_EXPONENT :=  V_CURRENCY_TABLE_RECORD.CURRENCY_EXPONENT;
        END IF;
        IF (V_COMPTEUR = 3)
        THEN
            FOR V_CARD_FEES_CREDIT_REC IN CUR_CARD_FEES_CREDIT(P_CLASS,P_CLASS_GROUP)
            LOOP
                IF (V_CARD_FEES_CREDIT_REC.FEES_AMOUNT_FIRST > 0)
                THEN
                    V_NBR_RECORD := V_NBR_RECORD + 1;
                    CASE V_CARD_FEES_CREDIT_REC.CARD_FEES_BILLING_PERIOD
                    WHEN 'Q' THEN V_FEES_AMOUNT := V_CARD_FEES_CREDIT_REC.FEES_AMOUNT_FIRST * 4;
                    WHEN 'M' THEN V_FEES_AMOUNT := V_CARD_FEES_CREDIT_REC.FEES_AMOUNT_FIRST * 12;
                    WHEN 'S' THEN V_FEES_AMOUNT := V_CARD_FEES_CREDIT_REC.FEES_AMOUNT_FIRST * 3;
                    WHEN 'Y' THEN V_FEES_AMOUNT := V_CARD_FEES_CREDIT_REC.FEES_AMOUNT_FIRST;
                    ELSE V_FEES_AMOUNT := 0;
                    END CASE;

                    RETURN_STATUS       :=  PCRD_CAI_GENERAL_TOOLS_1.CROSS_RATES_CONVERSION(GLOBAL_VARS.NETWORK_VISA,
                                                                                            V_CARD_FEES_CREDIT_REC.CURRENCY_CODE,
                                                                                            P_STAT_CURRENCY_TABLE_RECORD.CURRENCY_CODE,--MBH10012018 --P_BANK_RECORD.BANK_CURRENCY_CODE,
                                                                                            GLOBAL_VARS.RATE_ORIGIN_VISA,
                                                                                            GLOBAL_VARS.CONV_MODE_BAS_BUYING,
                                                                                            V_FEES_AMOUNT,
                                                                                            V_RATE_DATE,
                                                                                            V_RATE_ISO_STR_FORMAT,
                                                                                            V_FEES_AMOUNT_UC );

                    IF RETURN_STATUS = DECLARATION_CST.ERROR
                    THEN
                            V_ENV_INFO_TRACE.USER_MESSAGE := 'ERROR RETURNED BY PCRD_CAI_GENERAL_TOOLS_1.CROSS_RATES_CONVERSION:'         ||
                                                 'FROM CURRENCY :   '   ||  V_CARD_FEES_CREDIT_REC.CURRENCY_CODE    ||
                                                 'TO CURRENCY   :   '   ||  P_STAT_CURRENCY_TABLE_RECORD.CURRENCY_CODE;--MBH10012018 --P_BANK_RECORD.BANK_CURRENCY_CODE;
                            PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
                            RETURN DECLARATION_CST.ERROR;
                    ELSIF RETURN_STATUS = DECLARATION_CST.NOK
                    THEN
                        RETURN_STATUS       :=  PCRD_GENERAL_TOOLS.CONVERT_CURRENCY_AMOUNT( P_BANK_RECORD.BANK_CODE,
                                                                                            P_BANK_RECORD.BANK_CURRENCY_CODE,
                                                                                            V_CARD_FEES_CREDIT_REC.CURRENCY_CODE,
                                                                                            GLOBAL_VARS.RATE_ORIGIN_CENTER,
                                                                                            GLOBAL_VARS.CONV_MODE_BAS_BUYING,
                                                                                            V_STG_TRANS_CARD_CR_REC.USER_CURRENCY_CODE,--MBH10012018 --P_BANK_RECORD.BANK_CURRENCY_CODE,
                                                                                            V_FEES_AMOUNT,
                                                                                            P_BUSINESS_DATE,
                                                                                            NULL,
                                                                                            V_FEES_AMOUNT_UC,
                                                                                            V_CONVERSION_DATE,
                                                                                            V_RATE_ISO_STR_FORMAT,
                                                                                            V_MARKUP_AMOUNT,
                                                                                            FALSE,
                                                                                            NULL,
                                                                                            FALSE,
                                                                                            NULL );
                        IF RETURN_STATUS <> DECLARATION_CST.OK
                        THEN
                            V_ENV_INFO_TRACE.USER_MESSAGE    := 'ERROR CALLING PCRD_GENERAL_TOOLS.CONVERT_CURRENCY_AMOUNT '
                                                                ||' FROM CURRENCY : '           ||  V_CARD_FEES_CREDIT_REC.CURRENCY_CODE
                                                                ||'   TO  CURRENCY    :   '     ||  V_STG_TRANS_CARD_CR_REC.USER_CURRENCY_CODE;--MBH10012018 --P_BANK_RECORD.BANK_CURRENCY_CODE;
                            PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
                            RETURN(RETURN_STATUS);
                        END IF;
                    END IF;
                    V_STG_TRANS_CARD_CR_REC.ANUAL_FEES_LC := NVL(V_STG_TRANS_CARD_CR_REC.ANUAL_FEES_LC,0) + V_FEES_AMOUNT_UC;
                    V_STG_TRANS_CARD_CR_REC.ANUAL_FEES_UC := NVL(V_STG_TRANS_CARD_CR_REC.ANUAL_FEES_UC,0) + V_FEES_AMOUNT_UC; --MBH10012018
               END IF; --END IF OF (V_CARD_FEES_CREDIT_REC.FEES_AMOUNT_FIRST > 0)
            END LOOP;
            IF (V_NBR_RECORD > 0)
            THEN
                V_STG_TRANS_CARD_CR_REC.ANUAL_FEES_LC := NVL(V_STG_TRANS_CARD_CR_REC.ANUAL_FEES_LC,0) / V_NBR_RECORD;
                V_STG_TRANS_CARD_CR_REC.ANUAL_FEES_UC := NVL(V_STG_TRANS_CARD_CR_REC.ANUAL_FEES_UC,0) / V_NBR_RECORD; --MBH10012018
            END IF;
            BEGIN
                SELECT COUNT(1)
                INTO  V_STG_TRANS_CARD_CR_REC.NB_OF_MAILED_STATEMENTS
                FROM V_CR_TERM_VIEW T
                WHERE  STATEMENT_DATE    >=  P_START_DATE
                AND STATEMENT_DATE    <   P_END_DATE
                AND (   NBRTRA_CURBAL_PURCHASE      +
                        NBRTRA_CURBAL_CASH          +
                        NBRTRA_CURBAL_TRANSFER      +
                        NBRTRA_CURBAL_ECOM          +
                        NBRTRA_CURBAL_CHEQUE        +
                        NBRTRA_CURBAL_INTEREST      +
                        NBRTRA_CURBAL_INSTALMENT    +
                        NBRTRA_CURBAL_FEES          +
                        NBRTRA_CURBAL_TAX           +
                        NBRTRA_CURBAL_REFUND        +
                        NBRTRA_CURBAL_DEPOSIT       <>  0)
                AND  SHADOW_ACCOUNT_NBR IN
                    (
                    SELECT SHADOW_ACCOUNT_NBR
                    FROM  SHADOW_ACCOUNT A, CARD_PRODUCT B, CARD_TYPE C
                    WHERE A.PRIMARY_CARD_PRODUCT_CODE = B.PRODUCT_CODE
                    AND   A.BANK_CODE  = B.BANK_CODE
                    AND   A.BANK_CODE  = C.BANK_CODE
                    AND   B.NETWORK_CARD_TYPE = C.NETWORK_CARD_TYPE
                    AND   C.CLASS             = P_CLASS
                    AND   C.CLASS_GROUP       = P_CLASS_GROUP
                    AND   B.NETWORK_CODE = P_NETWORK_RECORD.NETWORK_CODE
                    AND   B.PRODUCT_TYPE = GLOBAL_VARS.CREDIT_CARD_TYPE)
                AND NOT EXISTS (SELECT 1
                                FROM CR_STATEMENT_HOLD
                                WHERE SHADOW_ACCOUNT_NBR = T.SHADOW_ACCOUNT_NBR
                                AND TRUNC(STATEMENT_DATE)  = TRUNC(T.STATEMENT_DATE));
                    EXCEPTION WHEN OTHERS
                    THEN
                        V_ENV_INFO_TRACE.USER_MESSAGE   := 'ERROR SELECT COUNT V_CR_TERM_VIEW';
                        PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
                    RETURN DECLARATION_CST.ERROR;
            END;
            BEGIN
                SELECT COUNT(1)
                INTO  V_STG_TRANS_CARD_CR_REC.NB_OF_MAILED_STAT_FINAN_CHARG
                FROM V_CR_TERM_VIEW T
                WHERE  STATEMENT_DATE    >=  P_START_DATE
                AND STATEMENT_DATE    <   P_END_DATE
                AND (CLOSING_BALANCE_FEES <> 0 OR CLOSING_BALANCE_INTEREST <> 0)
                AND  SHADOW_ACCOUNT_NBR IN
                    (
                    SELECT SHADOW_ACCOUNT_NBR
                    FROM  SHADOW_ACCOUNT A, CARD_PRODUCT B, CARD_TYPE C
                    WHERE A.PRIMARY_CARD_PRODUCT_CODE = B.PRODUCT_CODE
                    AND   A.BANK_CODE  = B.BANK_CODE
                    AND   A.BANK_CODE  = C.BANK_CODE
                    AND   B.NETWORK_CARD_TYPE = C.NETWORK_CARD_TYPE
                    AND   C.CLASS             = P_CLASS
                    AND   C.CLASS_GROUP       = P_CLASS_GROUP
                    AND   B.NETWORK_CODE = P_NETWORK_RECORD.NETWORK_CODE
                    AND   B.PRODUCT_TYPE = GLOBAL_VARS.CREDIT_CARD_TYPE)
                AND NOT EXISTS (SELECT 1
                                FROM CR_STATEMENT_HOLD
                                WHERE SHADOW_ACCOUNT_NBR = T.SHADOW_ACCOUNT_NBR
                                AND TRUNC(STATEMENT_DATE)  = TRUNC(T.STATEMENT_DATE));
                    EXCEPTION WHEN OTHERS
                    THEN
                        V_ENV_INFO_TRACE.USER_MESSAGE   := 'ERROR SELECT COUNT V_CR_TERM_VIEW';
                        PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
                    RETURN DECLARATION_CST.ERROR;
            END;
            BEGIN
                SELECT  COUNT(1)
                INTO V_STG_TRANS_CARD_CR_REC.NB_OF_INTERNATIONAL_ACCOUNT
                FROM SHADOW_ACCOUNT A, CARD_PRODUCT B, CARD_TYPE C
                WHERE A.BANK_CODE = B.BANK_CODE
                AND   A.BANK_CODE = C.BANK_CODE
                AND   A.PRIMARY_CARD_PRODUCT_CODE = B.PRODUCT_CODE
                AND   B.NETWORK_CARD_TYPE = C.NETWORK_CARD_TYPE
                AND   B.NETWORK_CODE = P_NETWORK_RECORD.NETWORK_CODE
                AND   A.BANK_CODE = P_BANK_RECORD.BANK_CODE
                AND   SUBSTR(SERVICE_CODE,1,1) IN ('1','2')
                AND   ADMINISTRATIVE_STATUS  <> '6'
                AND   C.CLASS             = P_CLASS
                AND   C.CLASS_GROUP       = P_CLASS_GROUP
                AND   PRODUCT_TYPE = GLOBAL_VARS.CREDIT_CARD_TYPE
                AND   AGREEMENT_DATE <= P_END_DATE     ;
                EXCEPTION WHEN OTHERS
                THEN
                    V_ENV_INFO_TRACE.USER_MESSAGE   := 'ERROR SELECT COUNT SHADOW_ACCOUNT';
                    PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
                RETURN DECLARATION_CST.ERROR;
            END;
            BEGIN
                SELECT  COUNT(1)
                INTO V_STG_TRANS_CARD_CR_REC.NB_OF_RESTRICTED_ACCOUNT
                FROM SHADOW_ACCOUNT A, CARD_PRODUCT B, CARD_TYPE C
                WHERE A.BANK_CODE = B.BANK_CODE
                AND   A.BANK_CODE = C.BANK_CODE
                AND   A.PRIMARY_CARD_PRODUCT_CODE = B.PRODUCT_CODE
                AND   B.NETWORK_CARD_TYPE = C.NETWORK_CARD_TYPE
                AND   B.NETWORK_CODE = P_NETWORK_RECORD.NETWORK_CODE
                AND   A.BANK_CODE = P_BANK_RECORD.BANK_CODE
                AND   SUBSTR(SERVICE_CODE,1,1) IN ('5','6','7')
                AND   ADMINISTRATIVE_STATUS  <> '6'
                AND   C.CLASS             = P_CLASS
                AND   C.CLASS_GROUP       = P_CLASS_GROUP
                AND   PRODUCT_TYPE = GLOBAL_VARS.CREDIT_CARD_TYPE
                AND   AGREEMENT_DATE <= P_END_DATE     ;
                EXCEPTION WHEN OTHERS
                THEN
                    V_ENV_INFO_TRACE.USER_MESSAGE   := 'ERROR SELECT COUNT SHADOW_ACCOUNT';
                    PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
                RETURN DECLARATION_CST.ERROR;
            END;
        END IF; --END IF OF (V_COMPTEUR = 3)

        RETURN_STATUS   :=  PCRD_STA_REPORTING_TOOLS_1_V3.PUT_ON_STG_CARD_STATISTICS_CR(V_STG_TRANS_CARD_CR_REC);
        IF (RETURN_STATUS <> DECLARATION_CST.OK)
        THEN
            V_ENV_INFO_TRACE.USER_MESSAGE := 'ERROR RETURNED BY PCRD_STA_REPORTING_TOOLS_1_V3.PUT_ON_STG_CARD_STATISTICS_CR STATISTICS_PERIOD = ' ||  V_STG_TRANS_CARD_CR_REC.STATISTICS_PERIOD ||
                                ', BANK_CODE = ' || V_STG_TRANS_CARD_CR_REC.BANK_CODE ||
                                ', NETWORK_CODE = ' || V_STG_TRANS_CARD_CR_REC.NETWORK_CODE ||
                                ', CLASS_GROUP = ' || V_STG_TRANS_CARD_CR_REC.CLASS_GROUP ||
                                 ', CLASS = ' || V_STG_TRANS_CARD_CR_REC.CLASS ;
            PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
            RETURN (DECLARATION_CST.ERROR);
        END IF;
        EXIT WHEN V_COMPTEUR = 3;
        V_COMPTEUR := V_COMPTEUR + 1;
        V_START_DATE    :=  ADD_MONTHS(V_START_DATE,1);
    END LOOP;
    RETURN(DECLARATION_CST.OK);
EXCEPTION WHEN OTHERS
THEN
    V_ENV_INFO_TRACE.USER_MESSAGE   :=  NULL;
    PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
    RETURN(DECLARATION_CST.NOK);
END PREP_STG_CARD_STATISTICS_CR;
-----------------------------------------------------------------------------------------------------------------------------------------
FUNCTION   PREP_QUARTERLY_CREDIT_REPORT  ( P_BANK_CODE                         IN      VARCHAR2,
                                            P_REPORT_CODE                       IN      VARCHAR2,
                                            P_PARAMETER_1                       IN      VARCHAR2,
                                            P_PARAMETER_2                       IN      VARCHAR2,
                                            P_PARAMETER_3                       IN      VARCHAR2,
                                            P_PARAMETER_4                       IN      VARCHAR2,
                                            P_PARAMETER_5                       IN      VARCHAR2,
                                            P_PARAMETER_6                       IN      VARCHAR2)
                                            RETURN PLS_INTEGER IS
/*P_PARAMETER_1==> QUARTER, P_PARAMETER_2==> YEAR*/
RETURN_STATUS                           PLS_INTEGER;
V_BUSINESS_DATE                         DATE ;
V_SWITCH_CUT_OFF_RECORD                 SWITCH_CUT_OFF%ROWTYPE;
V_LAST_CUTOFF_DATE                      VARCHAR2(12);
V_KEY_VAL                               RESSOURCE_BUNDLE.KEY_VAL%TYPE;
V_LANGUAGE_1                            PWC_BANK_REPORT_PARAMETERS.LANGUAGE_1%TYPE;
V_LANGUAGE_2                            PWC_BANK_REPORT_PARAMETERS.LANGUAGE_2%TYPE;
V_LANGUAGE_3                            PWC_BANK_REPORT_PARAMETERS.LANGUAGE_3%TYPE;
V_BANK_CODE                             BANK.BANK_CODE%TYPE;
V_CUTOFF_SEQUENCE                       CUTOFF_FOLLOW_UP.CUTOFF_SEQUENCE%TYPE;
V_NETWORK_RECORD                        NETWORK%ROWTYPE;
V_BANK_RECORD                           BANK%ROWTYPE;
V_START_DATE                            DATE;
V_END_DATE                              DATE;
V_BUILD_STATISTICS_PAR_REC              BUILD_STATISTICS_PARAM%ROWTYPE;
V_STAT_CURRENCY_TABLE_RECORD            CURRENCY_TABLE%ROWTYPE;
V_BANK_NETWORK_REC                      BANK_NETWORK%ROWTYPE; --MBH15122017

CURSOR  CUR_CARD_TYPE (P_BANK_CODE BANK.BANK_CODE%TYPE)  IS
SELECT  COUNT(1),BANK_CODE,NETWORK_CODE,CLASS,CLASS_GROUP
FROM    CARD_TYPE
WHERE   BANK_CODE = P_BANK_CODE
AND NETWORK_CODE = GLOBAL_VARS.NETWORK_VISA
AND CLASS_LEVEL  = '2' --SAB16072015
GROUP BY BANK_CODE,NETWORK_CODE,CLASS,CLASS_GROUP;

CURSOR CUR_PWC_BANK_REPORT_PARAMETERS   IS
    SELECT *
    FROM PWC_BANK_REPORT_PARAMETERS
    WHERE BANK_CODE = NVL(V_BANK_CODE,BANK_CODE)
    AND REPORT_CODE = P_REPORT_CODE;

BEGIN
      V_ENV_INFO_TRACE.USER_NAME      :=  GLOBAL_VARS.USER_NAME;
      V_ENV_INFO_TRACE.MODULE_CODE    :=  GLOBAL_VARS.ML_STATISTICS;
      V_ENV_INFO_TRACE.LANG           :=  GLOBAL_VARS.LANG;
      V_ENV_INFO_TRACE.PACKAGE_NAME   :=  $$PLSQL_UNIT;
      V_ENV_INFO_TRACE.FUNCTION_NAME  :=  'PREP_QUARTERLY_CREDIT_REPORT';

    IF P_BANK_CODE = 'ALL'
    THEN
        V_BANK_CODE := NULL;
    ELSE
        V_BANK_CODE := P_BANK_CODE;
    END IF;

    --START MBH21042017
    RETURN_STATUS := PCRD_REPORT_DATA_SOURCE.DELETE_BANK_DATA_REPORT('STG_AUTH_STATISTICS_CR',
                                                                     P_BANK_CODE);
    IF RETURN_STATUS <> DECLARATION_CST.OK
    THEN
        V_ENV_INFO_TRACE.USER_MESSAGE :=  'ERROR RETURNED BY PCRD_REPORT_DATA_SOURCE.DELETE_BANK_DATA_REPORT, ' ||
                                          ', TABLE = ' || 'STG_AUTH_STATISTICS_CR'        ||
                                          ', BANK_CODE = ' || P_BANK_CODE;
        PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
        RETURN(RETURN_STATUS);
    END IF;
    RETURN_STATUS := PCRD_REPORT_DATA_SOURCE.DELETE_BANK_DATA_REPORT('STG_CARD_STATISTICS_CR',
                                                                     P_BANK_CODE);
    IF RETURN_STATUS <> DECLARATION_CST.OK
    THEN
        V_ENV_INFO_TRACE.USER_MESSAGE :=  'ERROR RETURNED BY PCRD_REPORT_DATA_SOURCE.DELETE_BANK_DATA_REPORT, ' ||
                                          ', TABLE = ' || 'STG_CARD_STATISTICS_CR'        ||
                                          ', BANK_CODE = ' || P_BANK_CODE;
        PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
        RETURN(RETURN_STATUS);
    END IF;
    RETURN_STATUS := PCRD_REPORT_DATA_SOURCE.DELETE_BANK_DATA_REPORT('STG_FRAUD_STATISTICS_CR',
                                                                     P_BANK_CODE);
    IF RETURN_STATUS <> DECLARATION_CST.OK
    THEN
        V_ENV_INFO_TRACE.USER_MESSAGE :=  'ERROR RETURNED BY PCRD_REPORT_DATA_SOURCE.DELETE_BANK_DATA_REPORT, ' ||
                                          ', TABLE = ' || 'STG_FRAUD_STATISTICS_CR'        ||
                                          ', BANK_CODE = ' || P_BANK_CODE;
        PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
        RETURN(RETURN_STATUS);
    END IF;
    --END MBH21042017

    RETURN_STATUS := PCRD_GENERAL_TOOLS_1.GET_BUSINESS_DATE      ( V_SWITCH_CUT_OFF_RECORD,
                                                                   V_LAST_CUTOFF_DATE );
    IF RETURN_STATUS <> DECLARATION_CST.OK
    THEN
        V_ENV_INFO_TRACE.USER_MESSAGE   :=  'ERROR PCRD_GENERAL_TOOLS_1.GET_BUSINESS_DATE';
        PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
        RETURN(DECLARATION_CST.ERROR);
    END IF;
    V_BUSINESS_DATE := V_SWITCH_CUT_OFF_RECORD.LAST_CUTOFF_DATE_BO ;

    RETURN_STATUS   :=  PCRD_CUTOFF_FOLLOW_UP.READ_CUTOFF_SEQUENCE  (   V_BUSINESS_DATE,
                                                                        GLOBAL_VARS_CAI_1.STATISTICS,
                                                                        NULL,
                                                                        V_CUTOFF_SEQUENCE   );
    IF  RETURN_STATUS   <>  DECLARATION_CST.OK
    THEN
        V_ENV_INFO_TRACE.USER_MESSAGE :=  'ERROR RETURNED BY PCRD_CUTOFF_FOLLOW_UP.READ_CUTOFF_SEQUENCE';
        PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
        RETURN RETURN_STATUS;
    END IF;

    --START MBH10012018
    RETURN_STATUS := PCRD_STATISTIQUES_TOOLS_V3.UPLOAD_BANK_NETWORK;
    IF  (RETURN_STATUS <> DECLARATION_CST.OK)
    THEN
            V_ENV_INFO_TRACE.USER_MESSAGE   :=  'ERROR WHEN CALL PCRD_STATISTIQUES_TOOLS_V3.UPLOAD_BANK_NETWORK ';
            PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
            RETURN(RETURN_STATUS);
    END IF;
    /*RETURN_STATUS := PCRD_STATISTIQUES_0.GET_BUILD_STATISTICS_PARAM (   V_BUSINESS_DATE,
                                                                        V_BUILD_STATISTICS_PAR_REC );
    IF  (RETURN_STATUS <> DECLARATION_CST.OK)
    THEN
            V_ENV_INFO_TRACE.USER_MESSAGE   :=  'ERROR WHEN CALL PCRD_STATISTIQUES_0.GET_BUILD_STATISTICS_PARAM ';
            PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
            RETURN(RETURN_STATUS);
    END IF;

    RETURN_STATUS   :=  PCRD_GET_PARAM_GENERAL_ROWS.GET_CURRENCY_TABLE  (  V_BUILD_STATISTICS_PAR_REC.USER_CURRENCY,
                                                                           V_STAT_CURRENCY_TABLE_RECORD,
                                                                           TRUE);
    IF RETURN_STATUS <> DECLARATION_CST.OK
    THEN
            V_ENV_INFO_TRACE.USER_MESSAGE   :=  ':ERROR RETURNED BY PCRD_GET_PARAM_GENERAL_ROWS.GET_CURRENCY_TABLE'
                                            ||  ', STAT USER CURRENCY : ' || V_BUILD_STATISTICS_PAR_REC.USER_CURRENCY;
            PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
            RETURN(RETURN_STATUS);
    END IF;*/
    --END MBH10012018

    RETURN_STATUS   :=  PCRD_AUT_REPORTING_TOOLS_V3.GET_LANGUAGE_CODE(P_REPORT_CODE,
                                                                      V_LANGUAGE_1,
                                                                      V_LANGUAGE_2,
                                                                      V_LANGUAGE_3);
    IF RETURN_STATUS <> DECLARATION_CST.OK
    THEN
            V_ENV_INFO_TRACE.USER_MESSAGE := ':ERROR RETURNED BY PCRD_AUT_REPORTING_TOOLS_V3.GET_LANGUAGE_CODE:'
            ||'REPORT_CODE = '||P_REPORT_CODE;
            PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
            RETURN (DECLARATION_CST.ERROR);
    END IF;
    RETURN_STATUS   :=  PCRD_GET_PARAM_GENERAL_ROWS.GET_NETWORK  (  GLOBAL_VARS.NETWORK_VISA,
                                                                    V_NETWORK_RECORD);
    IF RETURN_STATUS <> DECLARATION_CST.OK
    THEN
        V_ENV_INFO_TRACE.USER_MESSAGE := ':ERROR RETURNED BY PCRD_GET_PARAM_GENERAL_ROWS.GET_NETWORK '
                                        || 'NETWORK CODE :' ||GLOBAL_VARS.NETWORK_VISA;
        PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
        RETURN (RETURN_STATUS);
    END IF;
    RETURN_STATUS   :=  PCRD_STA_REPORTING_TOOLS_V3.GET_START_DATE(P_PARAMETER_1,
                                                                    P_PARAMETER_2,
                                                                    V_START_DATE);
    IF (RETURN_STATUS <> DECLARATION_CST.OK)
    THEN
        V_ENV_INFO_TRACE.USER_MESSAGE := ':ERROR RETURNED BY PCRD_STA_REPORTING_TOOLS_V3.GET_START_DATE, QUARTER: '||P_PARAMETER_1||', YEAR: '||P_PARAMETER_2;
        PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
        RETURN (DECLARATION_CST.ERROR);
    END IF;
    RETURN_STATUS   :=  PCRD_STA_REPORTING_TOOLS_V3.GET_END_DATE(P_PARAMETER_1,
                                                                    P_PARAMETER_2,
                                                                    V_END_DATE);
    IF (RETURN_STATUS <> DECLARATION_CST.OK)
    THEN
        V_ENV_INFO_TRACE.USER_MESSAGE := ':ERROR RETURNED BY PCRD_STA_REPORTING_TOOLS_V3.GET_START_DATE, QUARTER: '||P_PARAMETER_1||', YEAR: '||P_PARAMETER_2;
        PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
        RETURN (DECLARATION_CST.ERROR);
    END IF;

    FOR V_PWC_BANK_REPORT_PARAM_REC IN CUR_PWC_BANK_REPORT_PARAMETERS
    LOOP
            --START MBH21042017
            DELETE FROM  STG_TRANSACTION_STATISTICS_CR
            WHERE  STATISTICS_PERIOD   BETWEEN V_START_DATE AND V_END_DATE
            AND    BANK_CODE = V_PWC_BANK_REPORT_PARAM_REC.BANK_CODE;
            --END MBH21042017

            RETURN_STATUS := PCRD_GET_PARAM_GENERAL_ROWS.GET_BANK ( V_PWC_BANK_REPORT_PARAM_REC.BANK_CODE,
                                                                    V_BANK_RECORD );
            IF RETURN_STATUS <> DECLARATION_CST.OK
            THEN
                V_ENV_INFO_TRACE.USER_MESSAGE   :=  'ERROR RETURNED BY PCRD_GET_PARAM_GENERAL_ROWS.GET_BANK'
                                                ||  ', BANK_CODE : '         || V_PWC_BANK_REPORT_PARAM_REC.BANK_CODE;
                PCRD_GENERAL_TOOLS.PUT_TRACES  (V_ENV_INFO_TRACE, $$PLSQL_LINE  );
                RETURN  RETURN_STATUS;
            END IF;
            --START MBH10012018
            RETURN_STATUS := PCRD_STATISTIQUES_TOOLS_V3.GET_BANK_NETWORK( V_PWC_BANK_REPORT_PARAM_REC.BANK_CODE,
                                                                          GLOBAL_VARS.NETWORK_VISA,
                                                                          V_BANK_NETWORK_REC);
            IF RETURN_STATUS <> DECLARATION_CST.OK
            THEN
                    V_ENV_INFO_TRACE.USER_MESSAGE := ':ERROR RETURNED BY PCRD_STATISTIQUES_TOOLS_V3.GET_BANK_NETWORK '
                                                    || 'BANK CODE :' ||V_PWC_BANK_REPORT_PARAM_REC.BANK_CODE
                                                    || 'NETWORK CODE :' ||GLOBAL_VARS.NETWORK_VISA;
                    PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
                    RETURN (RETURN_STATUS);
            END IF;
            RETURN_STATUS   :=  PCRD_GET_PARAM_GENERAL_ROWS.GET_CURRENCY_TABLE  (  V_BANK_NETWORK_REC.RECONCILIATION_CURRENCY,
                                                                                   V_STAT_CURRENCY_TABLE_RECORD,
                                                                                   TRUE);
            IF RETURN_STATUS <> DECLARATION_CST.OK
            THEN
                    V_ENV_INFO_TRACE.USER_MESSAGE   :=  'ERROR RETURNED BY PCRD_GET_PARAM_GENERAL_ROWS.GET_CURRENCY_TABLE'
                                                    ||  ', STAT CURRENCY CODE : ' || V_BANK_NETWORK_REC.RECONCILIATION_CURRENCY;
                    PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
                    RETURN(RETURN_STATUS);
            END IF;
            --END MBH10012018
            FOR V_CARD_TYPE_REC IN CUR_CARD_TYPE (V_PWC_BANK_REPORT_PARAM_REC.BANK_CODE)
            LOOP
                RETURN_STATUS :=    PCRD_STA_REPORTING_TOOLS_1_V3.PREP_STG_TRANSAC_STATISTICS_CR(V_BUSINESS_DATE,
                                                                                                V_CUTOFF_SEQUENCE,
                                                                                                V_LANGUAGE_1,
                                                                                                V_LANGUAGE_2,
                                                                                                V_LANGUAGE_3,
                                                                                                V_BANK_RECORD,
                                                                                                V_NETWORK_RECORD,
                                                                                                V_CARD_TYPE_REC.CLASS,
                                                                                                V_CARD_TYPE_REC.CLASS_GROUP,
                                                                                                V_START_DATE);
                IF RETURN_STATUS <> DECLARATION_CST.OK
                THEN
                    V_ENV_INFO_TRACE.USER_MESSAGE   :=  'ERROR RETURNED BY PCRD_STA_REPORTING_TOOLS_1_V3.PREP_STG_TRANSAC_STATISTICS_CR'
                                                    ||  ', BANK_CODE : '         || V_BANK_RECORD.BANK_CODE
                                                    ||  ', NETWORK_CODE : '         || V_NETWORK_RECORD.NETWORK_CODE
                                                    ||  ', CLASS : '         || V_CARD_TYPE_REC.CLASS
                                                    ||  ', CLASS_GROUP : '         || V_CARD_TYPE_REC.CLASS_GROUP
                                                    ||  ', START_DATE : '         || V_START_DATE;
                    PCRD_GENERAL_TOOLS.PUT_TRACES  (V_ENV_INFO_TRACE, $$PLSQL_LINE  );
                    ROLLBACK;
                    RETURN  RETURN_STATUS;
                END IF;
                RETURN_STATUS :=    PCRD_STA_REPORTING_TOOLS_1_V3.PREP_STG_CARD_STATISTICS_CR(V_BUSINESS_DATE,
                                                                                                V_CUTOFF_SEQUENCE,
                                                                                                V_LANGUAGE_1,
                                                                                                V_LANGUAGE_2,
                                                                                                V_LANGUAGE_3,
                                                                                                V_BANK_RECORD,
                                                                                                V_NETWORK_RECORD,
                                                                                                V_STAT_CURRENCY_TABLE_RECORD, --MBH10012018
                                                                                                V_CARD_TYPE_REC.CLASS,
                                                                                                V_CARD_TYPE_REC.CLASS_GROUP,
                                                                                                V_START_DATE,
                                                                                                V_END_DATE);
                IF RETURN_STATUS <> DECLARATION_CST.OK
                THEN
                    V_ENV_INFO_TRACE.USER_MESSAGE   :=  'ERROR RETURNED BY PCRD_STA_REPORTING_TOOLS_1_V3.PREP_STG_CARD_STATISTICS_CR'
                                                    ||  ', BANK_CODE : '         || V_BANK_RECORD.BANK_CODE
                                                    ||  ', NETWORK_CODE : '         || V_NETWORK_RECORD.NETWORK_CODE
                                                    ||  ', CLASS : '         || V_CARD_TYPE_REC.CLASS
                                                    ||  ', CLASS_GROUP : '         || V_CARD_TYPE_REC.CLASS_GROUP
                                                    ||  ', START_DATE : '         || V_START_DATE
                                                    ||  ', END_DATE : '         || V_END_DATE;
                    PCRD_GENERAL_TOOLS.PUT_TRACES  (V_ENV_INFO_TRACE, $$PLSQL_LINE  );
                    ROLLBACK;
                    RETURN  RETURN_STATUS;
                END IF;
                RETURN_STATUS :=    PCRD_STA_REPORTING_TOOLS_1_V3.PREP_STG_AUTH_STATISTICS_CR(V_BUSINESS_DATE,
                                                                                              V_CUTOFF_SEQUENCE,
                                                                                              V_LANGUAGE_1,
                                                                                              V_LANGUAGE_2,
                                                                                              V_LANGUAGE_3,
                                                                                              V_BANK_RECORD,
                                                                                              V_NETWORK_RECORD,
                                                                                              V_CARD_TYPE_REC.CLASS,
                                                                                              V_CARD_TYPE_REC.CLASS_GROUP,
                                                                                              V_START_DATE);
                IF RETURN_STATUS <> DECLARATION_CST.OK
                THEN
                    V_ENV_INFO_TRACE.USER_MESSAGE   :=  'ERROR RETURNED BY PCRD_STA_REPORTING_TOOLS_1_V3.PREP_STG_AUTH_STATISTICS_CR'
                                                    ||  ', BANK_CODE : '         || V_BANK_RECORD.BANK_CODE
                                                    ||  ', NETWORK_CODE : '         || V_NETWORK_RECORD.NETWORK_CODE
                                                    ||  ', CLASS : '         || V_CARD_TYPE_REC.CLASS
                                                    ||  ', CLASS_GROUP : '         || V_CARD_TYPE_REC.CLASS_GROUP
                                                    ||  ', START_DATE : '         || V_START_DATE;
                    PCRD_GENERAL_TOOLS.PUT_TRACES  (V_ENV_INFO_TRACE, $$PLSQL_LINE  );
                    ROLLBACK;
                    RETURN  RETURN_STATUS;
                END IF;
                RETURN_STATUS :=    PCRD_STA_REPORTING_TOOLS_1_V3.PREP_STG_FRAUD_STATISTICS_CR(V_BUSINESS_DATE,
                                                                                              V_CUTOFF_SEQUENCE,
                                                                                              V_LANGUAGE_1,
                                                                                              V_LANGUAGE_2,
                                                                                              V_LANGUAGE_3,
                                                                                              V_BANK_RECORD,
                                                                                              V_NETWORK_RECORD,
                                                                                              V_START_DATE,
                                                                                              V_END_DATE,
                                                                                              V_STAT_CURRENCY_TABLE_RECORD,
                                                                                              V_CARD_TYPE_REC.CLASS,
                                                                                              V_CARD_TYPE_REC.CLASS_GROUP);
                IF RETURN_STATUS <> DECLARATION_CST.OK
                THEN
                    V_ENV_INFO_TRACE.USER_MESSAGE   :=  'ERROR RETURNED BY PCRD_STA_REPORTING_TOOLS_1_V3.PREP_STG_FRAUD_STATISTICS_CR'
                                                    ||  ', BANK_CODE : '         || V_BANK_RECORD.BANK_CODE
                                                    ||  ', NETWORK_CODE : '         || V_NETWORK_RECORD.NETWORK_CODE
                                                    ||  ', END_DATE : '         || V_END_DATE
                                                    ||  ', START_DATE : '         || V_START_DATE
                                                    ||  ', CLASS_GROUP : '         || V_CARD_TYPE_REC.CLASS_GROUP
                                                    ||  ', CLASS : '         || V_CARD_TYPE_REC.CLASS;
                    PCRD_GENERAL_TOOLS.PUT_TRACES  (V_ENV_INFO_TRACE, $$PLSQL_LINE  );
                    ROLLBACK;
                    RETURN  RETURN_STATUS;
                END IF;
            END LOOP;
    END LOOP;
    COMMIT;
    RETURN(DECLARATION_CST.OK);
EXCEPTION WHEN OTHERS
THEN
    V_ENV_INFO_TRACE.USER_MESSAGE   :=  NULL;
    PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
    RETURN(DECLARATION_CST.NOK);
END PREP_QUARTERLY_CREDIT_REPORT;
--------------------------------------------------------------------------------------------------------------------------------------
FUNCTION        PUT_ON_STG_AUTH_STATISTICS_CR  (P_STG_AUTH_STATISTICS_CR_REC  IN           STG_AUTH_STATISTICS_CR%ROWTYPE)
                                            RETURN PLS_INTEGER IS
V_ENV_INFO_TRACE              GLOBAL_VARS.ENV_INFO_TRACE_TYPE:= NULL;
BEGIN
        V_ENV_INFO_TRACE.USER_NAME      :=  GLOBAL_VARS.USER_NAME;
        V_ENV_INFO_TRACE.MODULE_CODE    :=  GLOBAL_VARS.ML_STATISTICS;
        V_ENV_INFO_TRACE.PACKAGE_NAME   :=  $$PLSQL_UNIT;
        V_ENV_INFO_TRACE.FUNCTION_NAME  :=  'PUT_ON_STG_AUTH_STATISTICS_CR';
        V_ENV_INFO_TRACE.LANG           :=  GLOBAL_VARS.LANG;

        INSERT INTO STG_AUTH_STATISTICS_CR VALUES  P_STG_AUTH_STATISTICS_CR_REC;
        RETURN(DECLARATION_CST.OK);
EXCEPTION
WHEN    OTHERS  THEN
        V_ENV_INFO_TRACE.ERROR_CODE   := 'PWC-00016';
        V_ENV_INFO_TRACE.PARAM1       := 'STG_AUTH_STATISTICS_CR';
        V_ENV_INFO_TRACE.PARAM2       := 'STATISTICS_PERIOD = ' ||  P_STG_AUTH_STATISTICS_CR_REC.STATISTICS_PERIOD ||
                                        ', BANK_CODE = ' || P_STG_AUTH_STATISTICS_CR_REC.BANK_CODE ||
                                        ', NETWORK_CODE = ' || P_STG_AUTH_STATISTICS_CR_REC.NETWORK_CODE ||
                                        ', CLASS_GROUP = ' || P_STG_AUTH_STATISTICS_CR_REC.CLASS_GROUP ||
                                         ', CLASS = ' || P_STG_AUTH_STATISTICS_CR_REC.CLASS ;
        V_ENV_INFO_TRACE.USER_MESSAGE :=  NULL;
        PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
        RETURN( DECLARATION_CST.ERROR );
END PUT_ON_STG_AUTH_STATISTICS_CR;
--------------------------------------------------------------------------------------------------------------------------------------
FUNCTION   PREP_STG_AUTH_STATISTICS_CR   (  P_BUSINESS_DATE                     IN          DATE,
                                            P_CUTOFF_SEQUENCE                   IN          CUTOFF_FOLLOW_UP.CUTOFF_SEQUENCE%TYPE,
                                            P_LANGUAGE_1                        IN          PWC_BANK_REPORT_PARAMETERS.LANGUAGE_1%TYPE,
                                            P_LANGUAGE_2                        IN          PWC_BANK_REPORT_PARAMETERS.LANGUAGE_2%TYPE,
                                            P_LANGUAGE_3                        IN          PWC_BANK_REPORT_PARAMETERS.LANGUAGE_3%TYPE,
                                            P_BANK_RECORD                       IN          BANK%ROWTYPE,
                                            P_NETWORK_RECORD                    IN          NETWORK%ROWTYPE,
                                            P_CLASS                             IN          CARD_TYPE.CLASS%TYPE,
                                            P_CLASS_GROUP                       IN          CARD_TYPE.CLASS_GROUP%TYPE,
                                            P_START_DATE                        IN          DATE)
                                            RETURN PLS_INTEGER IS

RETURN_STATUS                           PLS_INTEGER;
V_START_DATE                            DATE;
V_COMPTEUR                              PLS_INTEGER;
V_STG_AUTH_STATISTICS_CR_REC            STG_AUTH_STATISTICS_CR%ROWTYPE;
V_KEY_VAL                               RESSOURCE_BUNDLE.KEY_VAL%TYPE;
V_MULTI_LANG_REPORT_V3                  MULTI_LANG_REPORT_V3;
V_BUNDLE_REPORT_V3                      BUNDLE_REPORT_V3;
V_ACTION_LIST_RECORD                    ACTION_LIST%ROWTYPE;
V_CURRENCY_TABLE_RECORD                 CURRENCY_TABLE%ROWTYPE;

CURSOR CUR_M_STAT_AUTH_ACTIVITY (P_BANK_CODE BANK.BANK_CODE%TYPE,P_NETWORK_CODE  NETWORK.NETWORK_CODE%TYPE, P_CLASS CARD_TYPE.CLASS%TYPE, P_CLASS_GROUP CARD_TYPE.CLASS_GROUP%TYPE, P_START_DATE    DATE) IS
    SELECT   *
    FROM     M_STAT_AUTH_ACTIVITY
    WHERE    STATISTICS_PERIOD         = TRUNC(TO_DATE(P_START_DATE),'MONTH')
    AND      ISSUING_BANK              = P_BANK_CODE
    AND      NETWORK_CODE              = P_NETWORK_CODE
    AND      CLASS                     = P_CLASS
    AND      CLASS_GROUP               = P_CLASS_GROUP
    AND      CARD_PRODUCT_TYPE         = GLOBAL_VARS.CREDIT_CARD_TYPE;

BEGIN
      V_ENV_INFO_TRACE.USER_NAME      :=  GLOBAL_VARS.USER_NAME;
      V_ENV_INFO_TRACE.MODULE_CODE    :=  GLOBAL_VARS.ML_STATISTICS;
      V_ENV_INFO_TRACE.LANG           :=  GLOBAL_VARS.LANG;
      V_ENV_INFO_TRACE.PACKAGE_NAME   :=  $$PLSQL_UNIT;
      V_ENV_INFO_TRACE.FUNCTION_NAME  :=  'PREP_STG_AUTH_STATISTICS_CR';

    V_START_DATE :=  TRUNC(P_START_DATE);
    V_COMPTEUR := 1;
    LOOP
        V_STG_AUTH_STATISTICS_CR_REC                                :=  NULL;
        V_STG_AUTH_STATISTICS_CR_REC.BANK_CODE                      :=  P_BANK_RECORD.BANK_CODE;
        V_STG_AUTH_STATISTICS_CR_REC.BANK_NAME                      :=  P_BANK_RECORD.BANK_NAME;
        V_STG_AUTH_STATISTICS_CR_REC.NETWORK_CODE                   :=  P_NETWORK_RECORD.NETWORK_CODE;
        V_STG_AUTH_STATISTICS_CR_REC.NETWORK_NAME                   :=  P_NETWORK_RECORD.NETWORK_LABEL;
        V_STG_AUTH_STATISTICS_CR_REC.BUSINESS_DATE                  :=  P_BUSINESS_DATE;
        V_STG_AUTH_STATISTICS_CR_REC.CUTOFF_ID                      :=  P_CUTOFF_SEQUENCE;
        V_STG_AUTH_STATISTICS_CR_REC.LANGUAGE_1                     :=  P_LANGUAGE_1;
        V_STG_AUTH_STATISTICS_CR_REC.LANGUAGE_2                     :=  P_LANGUAGE_2;
        V_STG_AUTH_STATISTICS_CR_REC.LANGUAGE_3                     :=  P_LANGUAGE_3;
        V_STG_AUTH_STATISTICS_CR_REC.STATISTICS_PERIOD              :=  V_START_DATE;
        V_STG_AUTH_STATISTICS_CR_REC.CLASS_GROUP                    :=  P_CLASS_GROUP;
        V_KEY_VAL := '';
        CASE    V_STG_AUTH_STATISTICS_CR_REC.CLASS_GROUP
        WHEN    'IN'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS_GROUP.IN';
        WHEN    'BU'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS_GROUP.BU';
        ELSE    V_KEY_VAL := 'NA';
        END CASE;
        V_BUNDLE_REPORT_V3 := PCRD_REPORTING_TOOLS_V3.NEW_BUNDLE_REPORT_V3;
        V_BUNDLE_REPORT_V3.KEY_VAL                      := V_KEY_VAL;
        V_BUNDLE_REPORT_V3.LANGUAGE_1                   := V_STG_AUTH_STATISTICS_CR_REC.LANGUAGE_1;
        V_BUNDLE_REPORT_V3.LANGUAGE_2                   := V_STG_AUTH_STATISTICS_CR_REC.LANGUAGE_2;
        V_BUNDLE_REPORT_V3.LANGUAGE_3                   := V_STG_AUTH_STATISTICS_CR_REC.LANGUAGE_3;
        RETURN_STATUS   :=  PCRD_REPORTING_TOOLS_V3.GET_REPORT_LABELS   ( V_BUNDLE_REPORT_V3 );
        IF RETURN_STATUS <> DECLARATION_CST.OK
        THEN
            V_ENV_INFO_TRACE.USER_MESSAGE   :=  'ERROR RETURNED BY PCRD_REPORTING_TOOLS_V3.GET_REPORT_LABELS'  ||
                                                ',BUNDLE : ' || PCRD_REPORTING_TOOLS_V3.BUNDLE ||
                                                ',KEY_VAL : ' || V_KEY_VAL;
            PCRD_GENERAL_TOOLS.PUT_TRACES  (V_ENV_INFO_TRACE, $$PLSQL_LINE  );
            RETURN  ( DECLARATION_CST.ERROR );
        END IF;
        V_STG_AUTH_STATISTICS_CR_REC.CLASS_GROUP_DESC_1             :=  V_BUNDLE_REPORT_V3.VALUE_1;
        V_STG_AUTH_STATISTICS_CR_REC.CLASS_GROUP_DESC_2             :=  V_BUNDLE_REPORT_V3.VALUE_2;
        V_STG_AUTH_STATISTICS_CR_REC.CLASS_GROUP_DESC_3             :=  V_BUNDLE_REPORT_V3.VALUE_3;
        V_STG_AUTH_STATISTICS_CR_REC.CLASS                          :=  P_CLASS;
        V_KEY_VAL := '';
        CASE    V_STG_AUTH_STATISTICS_CR_REC.CLASS
        WHEN    '01'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS.1';
        WHEN    '02'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS.2';
        WHEN    '03'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS.3';
        WHEN    '04'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS.4';
        WHEN    '05'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS.5';
        WHEN    '06'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS.6';
        ELSE    V_KEY_VAL := 'NA';
        END CASE;
        V_BUNDLE_REPORT_V3 := PCRD_REPORTING_TOOLS_V3.NEW_BUNDLE_REPORT_V3;
        V_BUNDLE_REPORT_V3.KEY_VAL                      := V_KEY_VAL;
        V_BUNDLE_REPORT_V3.LANGUAGE_1                   := V_STG_AUTH_STATISTICS_CR_REC.LANGUAGE_1;
        V_BUNDLE_REPORT_V3.LANGUAGE_2                   := V_STG_AUTH_STATISTICS_CR_REC.LANGUAGE_2;
        V_BUNDLE_REPORT_V3.LANGUAGE_3                   := V_STG_AUTH_STATISTICS_CR_REC.LANGUAGE_3;
        RETURN_STATUS   :=  PCRD_REPORTING_TOOLS_V3.GET_REPORT_LABELS   ( V_BUNDLE_REPORT_V3 );
        IF RETURN_STATUS <> DECLARATION_CST.OK
        THEN
            V_ENV_INFO_TRACE.USER_MESSAGE   :=  'ERROR RETURNED BY PCRD_REPORTING_TOOLS_V3.GET_REPORT_LABELS'  ||
                                                ',BUNDLE : ' || PCRD_REPORTING_TOOLS_V3.BUNDLE ||
                                                ',KEY_VAL : ' || V_KEY_VAL;
            PCRD_GENERAL_TOOLS.PUT_TRACES  (V_ENV_INFO_TRACE, $$PLSQL_LINE  );
            RETURN  ( DECLARATION_CST.ERROR );
        END IF;
        V_STG_AUTH_STATISTICS_CR_REC.CLASS_DESCRIPTION_1            :=  V_BUNDLE_REPORT_V3.VALUE_1;
        V_STG_AUTH_STATISTICS_CR_REC.CLASS_DESCRIPTION_2            :=  V_BUNDLE_REPORT_V3.VALUE_2;
        V_STG_AUTH_STATISTICS_CR_REC.CLASS_DESCRIPTION_3            :=  V_BUNDLE_REPORT_V3.VALUE_3;
        V_STG_AUTH_STATISTICS_CR_REC.NB_AUTHORIZATION               :=   0;
        V_STG_AUTH_STATISTICS_CR_REC.TRANSACTION_AMOUNT_LC          :=   0.;
        V_STG_AUTH_STATISTICS_CR_REC.TRANSACTION_AMOUNT_UC          :=   0.;
        V_STG_AUTH_STATISTICS_CR_REC.CASH_BACK_AMOUNT_LC            :=   0.;
        V_STG_AUTH_STATISTICS_CR_REC.CASH_BACK_AMOUNT_UC            :=   0.;
        V_STG_AUTH_STATISTICS_CR_REC.BILLING_AMOUNT_LC              :=   0.;
        V_STG_AUTH_STATISTICS_CR_REC.BILLING_AMOUNT_UC              :=   0.;
        V_STG_AUTH_STATISTICS_CR_REC.ISS_SETTLEMENT_AMOUNT_LC       :=   0.;
        V_STG_AUTH_STATISTICS_CR_REC.ISS_SETTLEMENT_AMOUNT_UC       :=   0.;
        V_STG_AUTH_STATISTICS_CR_REC.ACQ_SETTLEMENT_AMOUNT_LC       :=   0.;
        V_STG_AUTH_STATISTICS_CR_REC.ACQ_SETTLEMENT_AMOUNT_UC       :=   0.;
        V_STG_AUTH_STATISTICS_CR_REC.TRANSACTION_FEE_LC             :=   0.;
        V_STG_AUTH_STATISTICS_CR_REC.TRANSACTION_FEE_UC             :=   0.;
        V_STG_AUTH_STATISTICS_CR_REC.ISS_SETTLEMENT_FEE_LC          :=   0.;
        V_STG_AUTH_STATISTICS_CR_REC.ISS_SETTLEMENT_FEE_UC          :=   0.;
        V_STG_AUTH_STATISTICS_CR_REC.ACQ_SETTLEMENT_FEE_LC          :=   0.;
        V_STG_AUTH_STATISTICS_CR_REC.ACQ_SETTLEMENT_FEE_UC          :=   0.;
        V_STG_AUTH_STATISTICS_CR_REC.NBR_TRX_NOT_APPROVED           :=   0;
        V_STG_AUTH_STATISTICS_CR_REC.NBR_TRX_REFERRALS              :=   0;

        FOR V_M_STAT_AUTH_ACTIVITY_REC IN CUR_M_STAT_AUTH_ACTIVITY (P_BANK_RECORD.BANK_CODE,P_NETWORK_RECORD.NETWORK_CODE,P_CLASS,P_CLASS_GROUP,V_START_DATE)
        LOOP
            V_STG_AUTH_STATISTICS_CR_REC.LOCAL_CURRENCY_CODE            :=   V_M_STAT_AUTH_ACTIVITY_REC.LOCAL_CURRENCY_CODE;
            V_STG_AUTH_STATISTICS_CR_REC.LOCAL_CURRENCY_EXPONENT        :=   V_M_STAT_AUTH_ACTIVITY_REC.LOCAL_CURRENCY_EXP;
            V_STG_AUTH_STATISTICS_CR_REC.USER_CURRENCY_CODE             :=   V_M_STAT_AUTH_ACTIVITY_REC.USER_CURRENCY_CODE;
            V_STG_AUTH_STATISTICS_CR_REC.USER_CURRENCY_EXPONENT         :=   V_M_STAT_AUTH_ACTIVITY_REC.USER_CURRENCY_EXP;
            V_STG_AUTH_STATISTICS_CR_REC.NB_AUTHORIZATION               :=   V_STG_AUTH_STATISTICS_CR_REC.NB_AUTHORIZATION         + V_M_STAT_AUTH_ACTIVITY_REC.NB_AUTHORIZATION;
            V_STG_AUTH_STATISTICS_CR_REC.TRANSACTION_AMOUNT_LC          :=   V_STG_AUTH_STATISTICS_CR_REC.TRANSACTION_AMOUNT_LC    + V_M_STAT_AUTH_ACTIVITY_REC.TRANSACTION_AMOUNT_LC;
            V_STG_AUTH_STATISTICS_CR_REC.TRANSACTION_AMOUNT_UC          :=   V_STG_AUTH_STATISTICS_CR_REC.TRANSACTION_AMOUNT_UC    + V_M_STAT_AUTH_ACTIVITY_REC.TRANSACTION_AMOUNT_UC;
            V_STG_AUTH_STATISTICS_CR_REC.CASH_BACK_AMOUNT_LC            :=   V_STG_AUTH_STATISTICS_CR_REC.CASH_BACK_AMOUNT_LC      + V_M_STAT_AUTH_ACTIVITY_REC.CASH_BACK_AMOUNT_LC;
            V_STG_AUTH_STATISTICS_CR_REC.CASH_BACK_AMOUNT_UC            :=   V_STG_AUTH_STATISTICS_CR_REC.CASH_BACK_AMOUNT_UC      + V_M_STAT_AUTH_ACTIVITY_REC.CASH_BACK_AMOUNT_UC;
            V_STG_AUTH_STATISTICS_CR_REC.BILLING_AMOUNT_LC              :=   V_STG_AUTH_STATISTICS_CR_REC.BILLING_AMOUNT_LC        + V_M_STAT_AUTH_ACTIVITY_REC.BILLING_AMOUNT_LC;
            V_STG_AUTH_STATISTICS_CR_REC.BILLING_AMOUNT_UC              :=   V_STG_AUTH_STATISTICS_CR_REC.BILLING_AMOUNT_UC        + V_M_STAT_AUTH_ACTIVITY_REC.BILLING_AMOUNT_UC;
            V_STG_AUTH_STATISTICS_CR_REC.ISS_SETTLEMENT_AMOUNT_LC       :=   V_STG_AUTH_STATISTICS_CR_REC.ISS_SETTLEMENT_AMOUNT_LC + V_M_STAT_AUTH_ACTIVITY_REC.ISS_SETTLEMENT_AMOUNT_LC;
            V_STG_AUTH_STATISTICS_CR_REC.ISS_SETTLEMENT_AMOUNT_UC       :=   V_STG_AUTH_STATISTICS_CR_REC.ISS_SETTLEMENT_AMOUNT_UC + V_M_STAT_AUTH_ACTIVITY_REC.ISS_SETTLEMENT_AMOUNT_UC;
            V_STG_AUTH_STATISTICS_CR_REC.ACQ_SETTLEMENT_AMOUNT_LC       :=   V_STG_AUTH_STATISTICS_CR_REC.ACQ_SETTLEMENT_AMOUNT_LC + V_M_STAT_AUTH_ACTIVITY_REC.ACQ_SETTLEMENT_AMOUNT_LC;
            V_STG_AUTH_STATISTICS_CR_REC.ACQ_SETTLEMENT_AMOUNT_UC       :=   V_STG_AUTH_STATISTICS_CR_REC.ACQ_SETTLEMENT_AMOUNT_UC + V_M_STAT_AUTH_ACTIVITY_REC.ACQ_SETTLEMENT_AMOUNT_UC;

            RETURN_STATUS  := PCRD_GET_PARAM_GENERAL_ROWS.GET_ACTION_LIST ( V_M_STAT_AUTH_ACTIVITY_REC.ACTION_CODE,
                                                                            V_ACTION_LIST_RECORD);
            IF RETURN_STATUS  <> DECLARATION_CST.OK
            THEN
                V_ENV_INFO_TRACE.USER_MESSAGE   := 'ERROR RETURNED BY PCRD_GET_PARAM_GENERAL_ROWS.GET_ACTION_LIST ACTION_CODE :' || V_M_STAT_AUTH_ACTIVITY_REC.ACTION_CODE;
                PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
                --START MBH07042016
                --RETURN(DECLARATION_CST.NOK);
                V_ACTION_LIST_RECORD.CODE_ACTION := '909';
                V_ACTION_LIST_RECORD.ACTION_FLAG := 'D';
                --END MBH07042016
            END IF;

            IF (V_M_STAT_AUTH_ACTIVITY_REC.ACTION_CODE = '107')
            THEN
                V_STG_AUTH_STATISTICS_CR_REC.NBR_TRX_REFERRALS  := V_STG_AUTH_STATISTICS_CR_REC.NBR_TRX_REFERRALS + V_M_STAT_AUTH_ACTIVITY_REC.NB_AUTHORIZATION;
            END IF;
            IF (V_ACTION_LIST_RECORD.ACTION_FLAG <> 'A')
            THEN
                V_STG_AUTH_STATISTICS_CR_REC.NBR_TRX_NOT_APPROVED  := V_STG_AUTH_STATISTICS_CR_REC.NBR_TRX_NOT_APPROVED + V_M_STAT_AUTH_ACTIVITY_REC.NB_AUTHORIZATION;
            END IF;
        END LOOP;
        IF (V_STG_AUTH_STATISTICS_CR_REC.LOCAL_CURRENCY_CODE IS NULL)
        THEN
            RETURN_STATUS   :=  PCRD_GET_PARAM_GENERAL_ROWS.GET_CURRENCY_TABLE (P_BANK_RECORD.BANK_CURRENCY_CODE,
                                                                                V_CURRENCY_TABLE_RECORD,
                                                                                TRUE);
            IF RETURN_STATUS <> DECLARATION_CST.OK
            THEN
                    V_ENV_INFO_TRACE.USER_MESSAGE := ':ERROR RETURNED BY PCRD_GET_PARAM_GENERAL_ROWS.GET_CURRENCY_TABLE';
                    PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
                    RETURN(RETURN_STATUS);
            END IF;
            V_STG_AUTH_STATISTICS_CR_REC.LOCAL_CURRENCY_CODE     :=  V_CURRENCY_TABLE_RECORD.CURRENCY_CODE;
            V_STG_AUTH_STATISTICS_CR_REC.LOCAL_CURRENCY_EXPONENT :=  V_CURRENCY_TABLE_RECORD.CURRENCY_EXPONENT;
        END IF;
        RETURN_STATUS   :=  PCRD_STA_REPORTING_TOOLS_1_V3.PUT_ON_STG_AUTH_STATISTICS_CR(V_STG_AUTH_STATISTICS_CR_REC);
        IF (RETURN_STATUS <> DECLARATION_CST.OK)
        THEN
            V_ENV_INFO_TRACE.USER_MESSAGE := 'ERROR RETURNED BY PCRD_STA_REPORTING_TOOLS_1_V3.PUT_ON_STG_AUTH_STATISTICS_CR STATISTICS_PERIOD = ' ||  V_STG_AUTH_STATISTICS_CR_REC.STATISTICS_PERIOD ||
                                ', BANK_CODE = ' || V_STG_AUTH_STATISTICS_CR_REC.BANK_CODE ||
                                ', NETWORK_CODE = ' || V_STG_AUTH_STATISTICS_CR_REC.NETWORK_CODE ||
                                ', CLASS_GROUP = ' || V_STG_AUTH_STATISTICS_CR_REC.CLASS_GROUP ||
                                 ', CLASS = ' || V_STG_AUTH_STATISTICS_CR_REC.CLASS ;
            PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
            RETURN (DECLARATION_CST.ERROR);
        END IF;
        EXIT WHEN V_COMPTEUR = 3;
        V_COMPTEUR := V_COMPTEUR + 1;
        V_START_DATE    :=  ADD_MONTHS(V_START_DATE,1);
    END LOOP;
    RETURN(DECLARATION_CST.OK);
EXCEPTION WHEN OTHERS
THEN
    V_ENV_INFO_TRACE.USER_MESSAGE   :=  NULL;
    PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
    RETURN(DECLARATION_CST.NOK);
END PREP_STG_AUTH_STATISTICS_CR;
--------------------------------------------------------------------------------------------------------------------------------------
FUNCTION   PREP_STG_FRAUD_STATISTICS_CR   ( P_BUSINESS_DATE                     IN          DATE,
                                            P_CUTOFF_SEQUENCE                   IN          CUTOFF_FOLLOW_UP.CUTOFF_SEQUENCE%TYPE,
                                            P_LANGUAGE_1                        IN          PWC_BANK_REPORT_PARAMETERS.LANGUAGE_1%TYPE,
                                            P_LANGUAGE_2                        IN          PWC_BANK_REPORT_PARAMETERS.LANGUAGE_2%TYPE,
                                            P_LANGUAGE_3                        IN          PWC_BANK_REPORT_PARAMETERS.LANGUAGE_3%TYPE,
                                            P_BANK_RECORD                       IN          BANK%ROWTYPE,
                                            P_NETWORK_RECORD                    IN          NETWORK%ROWTYPE,
                                            P_START_DATE                        IN          DATE,
                                            P_END_DATE                          IN          DATE,
                                            P_STAT_CURRENCY_TABLE_RECORD        IN          CURRENCY_TABLE%ROWTYPE,
                                            P_CLASS                             IN          CARD_TYPE.CLASS%TYPE,
                                            P_CLASS_GROUP                       IN          CARD_TYPE.CLASS_GROUP%TYPE
                                            )
                                            RETURN PLS_INTEGER IS
RETURN_STATUS                           PLS_INTEGER;
V_COMPTEUR                              PLS_INTEGER;
V_STG_FRAUD_STATISTICS_CR_REC           STG_FRAUD_STATISTICS_CR%ROWTYPE;
V_KEY_VAL                               RESSOURCE_BUNDLE.KEY_VAL%TYPE;
V_BUNDLE_REPORT_V3                      BUNDLE_REPORT_V3;
TYPE ACCOUNTS_FRAUD_TAB IS TABLE OF     VARCHAR2(24) INDEX BY BINARY_INTEGER;
V_ACCOUNTS_FRAUD_TAB                    ACCOUNTS_FRAUD_TAB;
V_NOT_ALLOW_ACCOUNT_ADDITION            BOOLEAN := FALSE;
V_TABLE_COUNTER                         PLS_INTEGER:=0;
V_FRAUD_AMOUNT                          VISA_FRAUD_REPORTING.FRAUD_AMOUNT%TYPE:=0;
--V_FRAUD_AMOUNT_LC                       VISA_FRAUD_REPORTING.FRAUD_AMOUNT%TYPE:=0; --MBH10012018
V_FRAUD_AMOUNT_UC                       VISA_FRAUD_REPORTING.FRAUD_AMOUNT%TYPE:=0;
V_CONVERSION_DATE                       CONVERSION_RATE.RATE_DATE%TYPE;
V_RATE_ISO_STR_FORMAT                   VARCHAR2(9);
V_MARKUP_AMOUNT                         TRANSACTION_HIST.CONVERSION_FEES%TYPE;
V_CURRENCY_TABLE_RECORD                 CURRENCY_TABLE%ROWTYPE;
V_AMOUNT_RECOV_FRD_LOS                  CHARGEBACK.AMOUNT%TYPE:=0;
V_AMOUNT_RECOV_FRD_LOS_LC               CHARGEBACK.AMOUNT%TYPE:=0;
V_AMOUNT_RECOV_FRD_LOS_UC               CHARGEBACK.AMOUNT%TYPE:=0;
V_CARD_RECORD                           CARD%ROWTYPE;
V_CARD_PRODUCT_RECORD                   CARD_PRODUCT%ROWTYPE;
V_RATE_DATE                             DATE;
V_CARD_TYPE_RECORD                      CARD_TYPE%ROWTYPE;

CURSOR CUR_VISA_FRAUD_REPORTING
IS
SELECT *
FROM VISA_FRAUD_REPORTING
WHERE SOURCE_BANK = P_BANK_RECORD.BANK_CODE
AND   PROCESSING_DATE BETWEEN  P_START_DATE AND P_END_DATE;

CURSOR CUR_CHARGEBACK (V_MICROFILM_REF_NUMBER  IN VISA_FRAUD_REPORTING.ACQUIRER_REFERENCE_NUMBER%TYPE,
                        V_MICROFILM_REF_SEQ    IN VISA_FRAUD_REPORTING.SEQUENCE_NUMBER%TYPE)
IS
SELECT *
FROM    CHARGEBACK
WHERE   MICROFILM_REF_NUMBER    =  V_MICROFILM_REF_NUMBER
AND     MICROFILM_REF_SEQ       =   V_MICROFILM_REF_SEQ;


BEGIN
          V_ENV_INFO_TRACE.USER_NAME      :=  GLOBAL_VARS.USER_NAME;
          V_ENV_INFO_TRACE.MODULE_CODE    :=  GLOBAL_VARS.ML_STATISTICS;
          V_ENV_INFO_TRACE.LANG           :=  GLOBAL_VARS.LANG;
          V_ENV_INFO_TRACE.PACKAGE_NAME   :=  $$PLSQL_UNIT;
          V_ENV_INFO_TRACE.FUNCTION_NAME  :=  'PREP_STG_FRAUD_STATISTICS_CR';

        RETURN_STATUS   :=  PCRD_GET_PARAM_GENERAL_ROWS.GET_CURRENCY_TABLE  (   P_BANK_RECORD.BANK_CURRENCY_CODE,
                                                                                V_CURRENCY_TABLE_RECORD,
                                                                                TRUE);
        IF RETURN_STATUS <> DECLARATION_CST.OK
        THEN
                V_ENV_INFO_TRACE.USER_MESSAGE := ':ERROR RETURNED BY PCRD_GET_PARAM_GENERAL_ROWS.GET_CURRENCY_TABLE';
                PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
                RETURN(RETURN_STATUS);
        END IF;
        V_STG_FRAUD_STATISTICS_CR_REC                                :=  NULL;
        V_STG_FRAUD_STATISTICS_CR_REC.BUSINESS_DATE                  :=  P_BUSINESS_DATE;
        V_STG_FRAUD_STATISTICS_CR_REC.BANK_CODE                      :=  P_BANK_RECORD.BANK_CODE;
        V_STG_FRAUD_STATISTICS_CR_REC.BANK_NAME                      :=  P_BANK_RECORD.BANK_NAME;
        V_STG_FRAUD_STATISTICS_CR_REC.NETWORK_CODE                   :=  P_NETWORK_RECORD.NETWORK_CODE;
        V_STG_FRAUD_STATISTICS_CR_REC.NETWORK_NAME                   :=  P_NETWORK_RECORD.NETWORK_LABEL;
        V_STG_FRAUD_STATISTICS_CR_REC.CUTOFF_ID                      :=  P_CUTOFF_SEQUENCE;
        V_STG_FRAUD_STATISTICS_CR_REC.LANGUAGE_1                     :=  P_LANGUAGE_1;
        V_STG_FRAUD_STATISTICS_CR_REC.LANGUAGE_2                     :=  P_LANGUAGE_2;
        V_STG_FRAUD_STATISTICS_CR_REC.LANGUAGE_3                     :=  P_LANGUAGE_3;
        V_STG_FRAUD_STATISTICS_CR_REC.STATISTICS_PERIOD              :=  P_START_DATE;
        V_STG_FRAUD_STATISTICS_CR_REC.LOCAL_CURRENCY_CODE            :=  V_CURRENCY_TABLE_RECORD.CURRENCY_CODE;
        V_STG_FRAUD_STATISTICS_CR_REC.LOCAL_CURRENCY_EXPONENT        :=  V_CURRENCY_TABLE_RECORD.CURRENCY_EXPONENT;
        V_STG_FRAUD_STATISTICS_CR_REC.USER_CURRENCY_CODE             :=  P_STAT_CURRENCY_TABLE_RECORD.CURRENCY_CODE;
        V_STG_FRAUD_STATISTICS_CR_REC.USER_CURRENCY_EXPONENT         :=  P_STAT_CURRENCY_TABLE_RECORD.CURRENCY_EXPONENT;
        V_STG_FRAUD_STATISTICS_CR_REC.CLASS                          :=  P_CLASS;
        V_STG_FRAUD_STATISTICS_CR_REC.CLASS_GROUP                    :=  P_CLASS_GROUP;
        V_KEY_VAL := '';
        CASE    V_STG_FRAUD_STATISTICS_CR_REC.CLASS_GROUP
        WHEN    'IN'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS_GROUP.IN';
        WHEN    'BU'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS_GROUP.BU';
        ELSE    V_KEY_VAL := 'NA';
        END CASE;
        V_BUNDLE_REPORT_V3 := PCRD_REPORTING_TOOLS_V3.NEW_BUNDLE_REPORT_V3;
        V_BUNDLE_REPORT_V3.KEY_VAL                      := V_KEY_VAL;
        V_BUNDLE_REPORT_V3.LANGUAGE_1                   := V_STG_FRAUD_STATISTICS_CR_REC.LANGUAGE_1;
        V_BUNDLE_REPORT_V3.LANGUAGE_2                   := V_STG_FRAUD_STATISTICS_CR_REC.LANGUAGE_2;
        V_BUNDLE_REPORT_V3.LANGUAGE_3                   := V_STG_FRAUD_STATISTICS_CR_REC.LANGUAGE_3;
        RETURN_STATUS   :=  PCRD_REPORTING_TOOLS_V3.GET_REPORT_LABELS   ( V_BUNDLE_REPORT_V3 );
        IF RETURN_STATUS <> DECLARATION_CST.OK
        THEN
            V_ENV_INFO_TRACE.USER_MESSAGE   :=  'ERROR RETURNED BY PCRD_REPORTING_TOOLS_V3.GET_REPORT_LABELS'  ||
                                                ',BUNDLE : ' || PCRD_REPORTING_TOOLS_V3.BUNDLE ||
                                                ',KEY_VAL : ' || V_KEY_VAL;
            PCRD_GENERAL_TOOLS.PUT_TRACES  (V_ENV_INFO_TRACE, $$PLSQL_LINE  );
            RETURN  ( DECLARATION_CST.ERROR );
        END IF;
        V_STG_FRAUD_STATISTICS_CR_REC.CLASS_GROUP_DESC_1             :=  V_BUNDLE_REPORT_V3.VALUE_1;
        V_STG_FRAUD_STATISTICS_CR_REC.CLASS_GROUP_DESC_2             :=  V_BUNDLE_REPORT_V3.VALUE_2;
        V_STG_FRAUD_STATISTICS_CR_REC.CLASS_GROUP_DESC_3             :=  V_BUNDLE_REPORT_V3.VALUE_3;
        V_KEY_VAL := '';
        CASE    V_STG_FRAUD_STATISTICS_CR_REC.CLASS
        WHEN    '01'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS.1';
        WHEN    '02'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS.2';
        WHEN    '03'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS.3';
        WHEN    '04'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS.4';
        WHEN    '05'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS.5';
        WHEN    '06'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS.6';
        ELSE    V_KEY_VAL := 'NA';
        END CASE;
        V_BUNDLE_REPORT_V3 := PCRD_REPORTING_TOOLS_V3.NEW_BUNDLE_REPORT_V3;
        V_BUNDLE_REPORT_V3.KEY_VAL                      := V_KEY_VAL;
        V_BUNDLE_REPORT_V3.LANGUAGE_1                   := V_STG_FRAUD_STATISTICS_CR_REC.LANGUAGE_1;
        V_BUNDLE_REPORT_V3.LANGUAGE_2                   := V_STG_FRAUD_STATISTICS_CR_REC.LANGUAGE_2;
        V_BUNDLE_REPORT_V3.LANGUAGE_3                   := V_STG_FRAUD_STATISTICS_CR_REC.LANGUAGE_3;
        RETURN_STATUS   :=  PCRD_REPORTING_TOOLS_V3.GET_REPORT_LABELS   ( V_BUNDLE_REPORT_V3 );
        IF RETURN_STATUS <> DECLARATION_CST.OK
        THEN
            V_ENV_INFO_TRACE.USER_MESSAGE   :=  'ERROR RETURNED BY PCRD_REPORTING_TOOLS_V3.GET_REPORT_LABELS'  ||
                                                ',BUNDLE : ' || PCRD_REPORTING_TOOLS_V3.BUNDLE ||
                                                ',KEY_VAL : ' || V_KEY_VAL;
            PCRD_GENERAL_TOOLS.PUT_TRACES  (V_ENV_INFO_TRACE, $$PLSQL_LINE  );
            RETURN  ( DECLARATION_CST.ERROR );
        END IF;
        V_STG_FRAUD_STATISTICS_CR_REC.CLASS_DESCRIPTION_1            :=  V_BUNDLE_REPORT_V3.VALUE_1;
        V_STG_FRAUD_STATISTICS_CR_REC.CLASS_DESCRIPTION_2            :=  V_BUNDLE_REPORT_V3.VALUE_2;
        V_STG_FRAUD_STATISTICS_CR_REC.CLASS_DESCRIPTION_3            :=  V_BUNDLE_REPORT_V3.VALUE_3;
        V_STG_FRAUD_STATISTICS_CR_REC.NBR_ACCOUNT_FRAUD_LOSSES       :=   0.;
        V_STG_FRAUD_STATISTICS_CR_REC.NBR_ACCOUNT_CREDIT_LOSSES      :=   0.;
        V_STG_FRAUD_STATISTICS_CR_REC.TOT_AMOUNT_GROSS_FRD_LOS_LC    :=   0.;
        V_STG_FRAUD_STATISTICS_CR_REC.TOT_AMOUNT_RECOV_FRD_LOS_LC    :=   0.;
        V_STG_FRAUD_STATISTICS_CR_REC.TOT_AMOUNT_GROS_CRDT_LOS_LC    :=   0.;
        V_STG_FRAUD_STATISTICS_CR_REC.TOT_AMOUNT_RECO_CRDT_LOS_LC    :=   0.;
        V_STG_FRAUD_STATISTICS_CR_REC.TOT_AMOUNT_GROSS_FRD_LOS_UC    :=   0.;
        V_STG_FRAUD_STATISTICS_CR_REC.TOT_AMOUNT_RECOV_FRD_LOS_UC    :=   0.;
        V_STG_FRAUD_STATISTICS_CR_REC.TOT_AMOUNT_GROS_CRDT_LOS_UC    :=   0.;
        V_STG_FRAUD_STATISTICS_CR_REC.TOT_AMOUNT_RECO_CRDT_LOS_UC    :=   0.;
        FOR V_CUR_VISA_FRAUD_REPORTING_REC   IN CUR_VISA_FRAUD_REPORTING
        LOOP
            RETURN_STATUS  :=  PCRD_GET_DATA_CARD_ROWS.GET_CARD (   RTRIM(V_CUR_VISA_FRAUD_REPORTING_REC.ACCOUNT_NUMBER||V_CUR_VISA_FRAUD_REPORTING_REC.ACCOUNT_NUMBER_EXTENSION),
                                                                    V_CARD_RECORD);
            IF      RETURN_STATUS = DECLARATION_CST.ERROR
            THEN
                V_ENV_INFO_TRACE.USER_MESSAGE   :=  'ERROR RETURNED BY PCRD_GET_DATA_CARD_ROWS.GET_CARD'
                                                /*||  ', CARD_NUMBER : ' || P_CARD_NUMBER*/;
                PCRD_GENERAL_TOOLS.PUT_TRACES  (V_ENV_INFO_TRACE, 210 );
                RETURN  RETURN_STATUS;

            ELSIF   RETURN_STATUS = DECLARATION_CST.NOK
            THEN
                V_ENV_INFO_TRACE.USER_MESSAGE   :=  'CARD DOESNT EXIST'
                                                /*||  ', CARD_NUMBER : ' || P_CARD_NUMBER*/;
                PCRD_GENERAL_TOOLS.PUT_TRACES  (V_ENV_INFO_TRACE, 120 );
                RETURN  RETURN_STATUS;
            END IF;
            RETURN_STATUS := PCRD_GET_PARAM_CARD_ROWS.GET_CARD_PRODUCT (V_CARD_RECORD.BANK_CODE,
                                                                        V_CARD_RECORD.CARD_PRODUCT_CODE,
                                                                        V_CARD_PRODUCT_RECORD,
                                                                        TRUE );
            IF  (RETURN_STATUS <> DECLARATION_CST.OK)
            THEN
                    V_ENV_INFO_TRACE.USER_MESSAGE   :=  'ERROR WHEN CALL PCRD_GET_PARAM_CARD_ROWS.GET_CARD_PRODUCT '
                                                        || 'BANK :'         ||V_CARD_RECORD.BANK_CODE
                                                        || ' ,PRODUCT :'    ||V_CARD_RECORD.CARD_PRODUCT_CODE;
                    PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
                    RETURN(DECLARATION_CST.ERROR);
            END IF;
            RETURN_STATUS   :=  PCRD_GET_PARAM_CARD_ROWS.GET_CARD_TYPE  (   V_CARD_PRODUCT_RECORD.NETWORK_CODE,
                                                                            V_CARD_PRODUCT_RECORD.BANK_CODE,
                                                                            V_CARD_PRODUCT_RECORD.NETWORK_CARD_TYPE,
                                                                            V_CARD_TYPE_RECORD);

            IF RETURN_STATUS <> DECLARATION_CST.OK
            THEN
                    V_ENV_INFO_TRACE.USER_MESSAGE := ':ERROR RETURNED BY PCRD_GET_PARAM_CARD_ROWS.GET_CARD_TYPE   '
                                                    ||V_CARD_PRODUCT_RECORD.NETWORK_CODE
                                                    ||':'||
                                                    V_CARD_PRODUCT_RECORD.NETWORK_CARD_TYPE
                                                    ||':'||
                                                    V_CARD_PRODUCT_RECORD.BANK_CODE;
                    PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
                    RETURN(RETURN_STATUS);
            END IF;
            IF (V_CARD_PRODUCT_RECORD.PRODUCT_TYPE = GLOBAL_VARS.CREDIT_CARD_TYPE AND
                V_CARD_TYPE_RECORD.CLASS = P_CLASS AND
                V_CARD_TYPE_RECORD.CLASS = P_CLASS_GROUP)
            THEN
                V_NOT_ALLOW_ACCOUNT_ADDITION   := FALSE;
                IF V_ACCOUNTS_FRAUD_TAB.COUNT > 0
                THEN
                    FOR I IN V_ACCOUNTS_FRAUD_TAB.FIRST..V_ACCOUNTS_FRAUD_TAB.LAST
                    LOOP
                        IF V_ACCOUNTS_FRAUD_TAB(I) = V_CUR_VISA_FRAUD_REPORTING_REC.ACCOUNT_NUMBER
                        THEN
                            V_NOT_ALLOW_ACCOUNT_ADDITION := TRUE;
                            EXIT;
                        END IF;
                    END LOOP;
                    IF V_NOT_ALLOW_ACCOUNT_ADDITION = FALSE
                    THEN
                        V_ACCOUNTS_FRAUD_TAB(V_ACCOUNTS_FRAUD_TAB.LAST + 1) := V_CUR_VISA_FRAUD_REPORTING_REC.ACCOUNT_NUMBER;
                        V_STG_FRAUD_STATISTICS_CR_REC.NBR_ACCOUNT_FRAUD_LOSSES := V_STG_FRAUD_STATISTICS_CR_REC.NBR_ACCOUNT_FRAUD_LOSSES + 1;
                    END IF;
                ELSE
                    V_ACCOUNTS_FRAUD_TAB(0) := V_CUR_VISA_FRAUD_REPORTING_REC.ACCOUNT_NUMBER;
                    V_STG_FRAUD_STATISTICS_CR_REC.NBR_ACCOUNT_FRAUD_LOSSES := V_STG_FRAUD_STATISTICS_CR_REC.NBR_ACCOUNT_FRAUD_LOSSES + 1;
                    V_NOT_ALLOW_ACCOUNT_ADDITION := FALSE;
                END IF;
                V_FRAUD_AMOUNT := NVL(V_CUR_VISA_FRAUD_REPORTING_REC.FRAUD_AMOUNT,0.);
                IF (V_FRAUD_AMOUNT <> 0.)
                THEN
                    RETURN_STATUS       :=  PCRD_CAI_GENERAL_TOOLS_1.CROSS_RATES_CONVERSION(GLOBAL_VARS.NETWORK_VISA,
                                                                                            V_CUR_VISA_FRAUD_REPORTING_REC.FRAUD_CURRENCY_CODE,
                                                                                            V_STG_FRAUD_STATISTICS_CR_REC.USER_CURRENCY_CODE,--MBH10012018 --P_BANK_RECORD.BANK_CURRENCY_CODE,
                                                                                            GLOBAL_VARS.RATE_ORIGIN_VISA,
                                                                                            GLOBAL_VARS.CONV_MODE_BAS_BUYING,
                                                                                            V_FRAUD_AMOUNT,
                                                                                            V_RATE_DATE,
                                                                                            V_RATE_ISO_STR_FORMAT,
                                                                                            V_FRAUD_AMOUNT_UC );

                    IF RETURN_STATUS = DECLARATION_CST.ERROR
                    THEN
                            V_ENV_INFO_TRACE.USER_MESSAGE := 'ERROR RETURNED BY PCRD_CAI_GENERAL_TOOLS_1.CROSS_RATES_CONVERSION:'         ||
                                                 'FROM CURRENCY :   '   ||  V_CUR_VISA_FRAUD_REPORTING_REC.FRAUD_CURRENCY_CODE   ||
                                                 'TO CURRENCY   :   '   ||  V_STG_FRAUD_STATISTICS_CR_REC.USER_CURRENCY_CODE;--MBH10012018 --P_BANK_RECORD.BANK_CURRENCY_CODE;
                            PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
                            RETURN DECLARATION_CST.ERROR;
                    ELSIF RETURN_STATUS = DECLARATION_CST.NOK
                    THEN
                        RETURN_STATUS       :=  PCRD_GENERAL_TOOLS.CONVERT_CURRENCY_AMOUNT( P_BANK_RECORD.BANK_CODE,
                                                                                            P_BANK_RECORD.BANK_CURRENCY_CODE,
                                                                                            V_CUR_VISA_FRAUD_REPORTING_REC.FRAUD_CURRENCY_CODE,
                                                                                            GLOBAL_VARS.RATE_ORIGIN_CENTER,
                                                                                            GLOBAL_VARS.CONV_MODE_BAS_BUYING,
                                                                                            V_STG_FRAUD_STATISTICS_CR_REC.USER_CURRENCY_CODE,--MBH10012018 --P_BANK_RECORD.BANK_CURRENCY_CODE,
                                                                                            V_FRAUD_AMOUNT,
                                                                                            P_BUSINESS_DATE,
                                                                                            NULL,
                                                                                            V_FRAUD_AMOUNT_UC,
                                                                                            V_CONVERSION_DATE,
                                                                                            V_RATE_ISO_STR_FORMAT,
                                                                                            V_MARKUP_AMOUNT,
                                                                                            FALSE,
                                                                                            NULL,
                                                                                            FALSE,
                                                                                            NULL );
                        IF RETURN_STATUS <> DECLARATION_CST.OK
                        THEN
                            V_ENV_INFO_TRACE.USER_MESSAGE    := 'ERROR CALLING PCRD_GENERAL_TOOLS.CONVERT_CURRENCY_AMOUNT '
                                                                ||' FROM CURRENCY : '           ||  V_CUR_VISA_FRAUD_REPORTING_REC.FRAUD_CURRENCY_CODE
                                                                ||'   TO  CURRENCY    :   '     ||  V_STG_FRAUD_STATISTICS_CR_REC.USER_CURRENCY_CODE;--MBH10012018 --P_BANK_RECORD.BANK_CURRENCY_CODE;
                            PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
                            RETURN(RETURN_STATUS);
                        END IF;
                    END IF;
                    V_STG_FRAUD_STATISTICS_CR_REC.TOT_AMOUNT_GROSS_FRD_LOS_LC    :=   NVL(V_STG_FRAUD_STATISTICS_CR_REC.TOT_AMOUNT_GROSS_FRD_LOS_LC,0) + V_FRAUD_AMOUNT_UC;
                END IF;
                FOR V_CUR_CHARGEBACK_REC   IN CUR_CHARGEBACK (V_CUR_VISA_FRAUD_REPORTING_REC.ACQUIRER_REFERENCE_NUMBER, V_CUR_VISA_FRAUD_REPORTING_REC.SEQUENCE_NUMBER)
                LOOP
                   V_AMOUNT_RECOV_FRD_LOS      := 0.;
                   V_AMOUNT_RECOV_FRD_LOS_LC   := 0.;
                   V_AMOUNT_RECOV_FRD_LOS_UC   := 0.;
                   IF (V_CUR_CHARGEBACK_REC.REVERSAL_FLAG = 'R')
                   THEN
                       V_AMOUNT_RECOV_FRD_LOS  := V_AMOUNT_RECOV_FRD_LOS - V_CUR_CHARGEBACK_REC.AMOUNT;
                   ELSE
                       V_AMOUNT_RECOV_FRD_LOS  := V_AMOUNT_RECOV_FRD_LOS + V_CUR_CHARGEBACK_REC.AMOUNT;
                   END IF;
                   IF (V_AMOUNT_RECOV_FRD_LOS <> 0.)
                   THEN
                        RETURN_STATUS       :=  PCRD_CAI_GENERAL_TOOLS_1.CROSS_RATES_CONVERSION(GLOBAL_VARS.NETWORK_VISA,
                                                                                            V_CUR_VISA_FRAUD_REPORTING_REC.FRAUD_CURRENCY_CODE,
                                                                                            V_STG_FRAUD_STATISTICS_CR_REC.USER_CURRENCY_CODE,--MBH10012018 --P_BANK_RECORD.BANK_CURRENCY_CODE,
                                                                                            GLOBAL_VARS.RATE_ORIGIN_VISA,
                                                                                            GLOBAL_VARS.CONV_MODE_BAS_BUYING,
                                                                                            V_AMOUNT_RECOV_FRD_LOS,
                                                                                            V_RATE_DATE,
                                                                                            V_RATE_ISO_STR_FORMAT,
                                                                                            V_AMOUNT_RECOV_FRD_LOS_LC );

                        IF RETURN_STATUS = DECLARATION_CST.ERROR
                        THEN
                                V_ENV_INFO_TRACE.USER_MESSAGE := 'ERROR RETURNED BY PCRD_CAI_GENERAL_TOOLS_1.CROSS_RATES_CONVERSION:'         ||
                                                     'FROM CURRENCY :   '   ||  V_CUR_VISA_FRAUD_REPORTING_REC.FRAUD_CURRENCY_CODE   ||
                                                     'TO CURRENCY   :   '   ||  V_STG_FRAUD_STATISTICS_CR_REC.USER_CURRENCY_CODE;--MBH10012018 --P_BANK_RECORD.BANK_CURRENCY_CODE;
                                PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
                                RETURN DECLARATION_CST.ERROR;
                        ELSIF RETURN_STATUS = DECLARATION_CST.NOK
                        THEN
                            RETURN_STATUS       :=  PCRD_GENERAL_TOOLS.CONVERT_CURRENCY_AMOUNT( P_BANK_RECORD.BANK_CODE,
                                                                                                P_BANK_RECORD.BANK_CURRENCY_CODE,
                                                                                                V_CUR_VISA_FRAUD_REPORTING_REC.FRAUD_CURRENCY_CODE,
                                                                                                GLOBAL_VARS.RATE_ORIGIN_CENTER,
                                                                                                GLOBAL_VARS.CONV_MODE_BAS_BUYING,
                                                                                                V_STG_FRAUD_STATISTICS_CR_REC.USER_CURRENCY_CODE,--MBH10012018 --P_BANK_RECORD.BANK_CURRENCY_CODE,
                                                                                                V_AMOUNT_RECOV_FRD_LOS,
                                                                                                P_BUSINESS_DATE,
                                                                                                NULL,
                                                                                                V_AMOUNT_RECOV_FRD_LOS_LC,
                                                                                                V_CONVERSION_DATE,
                                                                                                V_RATE_ISO_STR_FORMAT,
                                                                                                V_MARKUP_AMOUNT,
                                                                                                FALSE,
                                                                                                NULL,
                                                                                                FALSE,
                                                                                                NULL );
                            IF RETURN_STATUS <> DECLARATION_CST.OK
                            THEN
                                V_ENV_INFO_TRACE.USER_MESSAGE    := 'ERROR CALLING PCRD_GENERAL_TOOLS.CONVERT_CURRENCY_AMOUNT '
                                                                    ||' FROM CURRENCY : '           ||  V_CUR_VISA_FRAUD_REPORTING_REC.FRAUD_CURRENCY_CODE
                                                                    ||'   TO  CURRENCY    :   '     ||  V_STG_FRAUD_STATISTICS_CR_REC.USER_CURRENCY_CODE;--MBH10012018 --P_BANK_RECORD.BANK_CURRENCY_CODE;
                                PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
                                RETURN(RETURN_STATUS);
                            END IF;
                            V_STG_FRAUD_STATISTICS_CR_REC.TOT_AMOUNT_RECOV_FRD_LOS_LC    :=   NVL(V_STG_FRAUD_STATISTICS_CR_REC.TOT_AMOUNT_RECOV_FRD_LOS_LC,0) + V_FRAUD_AMOUNT_UC;
                        END IF;
                   END IF;
                END LOOP;
             END IF;
        END LOOP;
        RETURN_STATUS   :=  PCRD_STA_REPORTING_TOOLS_1_V3.PUT_ON_STG_FRAUD_STATISTICS_CR(V_STG_FRAUD_STATISTICS_CR_REC);
        IF (RETURN_STATUS <> DECLARATION_CST.OK)
        THEN
            V_ENV_INFO_TRACE.USER_MESSAGE := 'ERROR RETURNED BY PCRD_STA_REPORTING_TOOLS_1_V3.PUT_ON_STG_FRAUD_STATISTICS_CR, STATISTICS_PERIOD = ' ||  V_STG_FRAUD_STATISTICS_CR_REC.STATISTICS_PERIOD ||
                                ', BANK_CODE = ' || V_STG_FRAUD_STATISTICS_CR_REC.BANK_CODE ||
                                ', NETWORK_CODE = ' || V_STG_FRAUD_STATISTICS_CR_REC.NETWORK_CODE ;
            PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
            RETURN (DECLARATION_CST.ERROR);
        END IF;
    RETURN(DECLARATION_CST.OK);
EXCEPTION WHEN OTHERS
THEN
    V_ENV_INFO_TRACE.USER_MESSAGE   :=  NULL;
    PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
    RETURN(DECLARATION_CST.NOK);
END PREP_STG_FRAUD_STATISTICS_CR;
--------------------------------------------------------------------------------------------------------------------------------------
FUNCTION        PUT_ON_STG_FRAUD_STATISTICS_CR  (P_STG_FRAUD_STATISTICS_CR_REC  IN           STG_FRAUD_STATISTICS_CR%ROWTYPE)
                                            RETURN PLS_INTEGER IS
V_ENV_INFO_TRACE              GLOBAL_VARS.ENV_INFO_TRACE_TYPE:= NULL;
BEGIN
        V_ENV_INFO_TRACE.USER_NAME      :=  GLOBAL_VARS.USER_NAME;
        V_ENV_INFO_TRACE.MODULE_CODE    :=  GLOBAL_VARS.ML_STATISTICS;
        V_ENV_INFO_TRACE.PACKAGE_NAME   :=  $$PLSQL_UNIT;
        V_ENV_INFO_TRACE.FUNCTION_NAME  :=  'PUT_ON_STG_FRAUD_STATISTICS_CR';
        V_ENV_INFO_TRACE.LANG           :=  GLOBAL_VARS.LANG;

        INSERT INTO STG_FRAUD_STATISTICS_CR VALUES  P_STG_FRAUD_STATISTICS_CR_REC;
        RETURN(DECLARATION_CST.OK);
EXCEPTION
WHEN    OTHERS  THEN
        V_ENV_INFO_TRACE.ERROR_CODE   := 'PWC-00016';
        V_ENV_INFO_TRACE.PARAM1       := 'STG_FRAUD_STATISTICS_CR';
        V_ENV_INFO_TRACE.PARAM2       := 'STATISTICS_PERIOD = ' ||  P_STG_FRAUD_STATISTICS_CR_REC.STATISTICS_PERIOD ||
                                        ', BANK_CODE = ' || P_STG_FRAUD_STATISTICS_CR_REC.BANK_CODE ||
                                        ', NETWORK_CODE = ' || P_STG_FRAUD_STATISTICS_CR_REC.NETWORK_CODE ;
        V_ENV_INFO_TRACE.USER_MESSAGE :=  NULL;
        PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
        RETURN( DECLARATION_CST.ERROR );
END PUT_ON_STG_FRAUD_STATISTICS_CR;
--------------------------------------------------------------------------------------------------------------------------------------
FUNCTION        PUT_ON_STG_ACCOUNT_DELINQ_CR  (P_STG_STG_ACCT_DELINQ_CR_REC  IN           STG_ACCOUNT_DELINQUENCIES_CR%ROWTYPE)
                                                RETURN PLS_INTEGER IS
V_ENV_INFO_TRACE              GLOBAL_VARS.ENV_INFO_TRACE_TYPE:= NULL;
BEGIN
        V_ENV_INFO_TRACE.USER_NAME      :=  GLOBAL_VARS.USER_NAME;
        V_ENV_INFO_TRACE.MODULE_CODE    :=  GLOBAL_VARS.ML_STATISTICS;
        V_ENV_INFO_TRACE.PACKAGE_NAME   :=  $$PLSQL_UNIT;
        V_ENV_INFO_TRACE.FUNCTION_NAME  :=  'PUT_ON_STG_ACCOUNT_DELINQ_CR';
        V_ENV_INFO_TRACE.LANG           :=  GLOBAL_VARS.LANG;

        INSERT INTO STG_ACCOUNT_DELINQUENCIES_CR VALUES  P_STG_STG_ACCT_DELINQ_CR_REC;
        RETURN(DECLARATION_CST.OK);
EXCEPTION
WHEN    OTHERS  THEN
        V_ENV_INFO_TRACE.ERROR_CODE   := 'PWC-00016';
        V_ENV_INFO_TRACE.PARAM1       := 'STG_AUTH_STATISTICS_CR';
        V_ENV_INFO_TRACE.PARAM2       := 'STATISTICS_PERIOD = ' ||  P_STG_STG_ACCT_DELINQ_CR_REC.STATISTICS_PERIOD ||
                                        ', BANK_CODE = ' || P_STG_STG_ACCT_DELINQ_CR_REC.BANK_CODE ||
                                        ', NETWORK_CODE = ' || P_STG_STG_ACCT_DELINQ_CR_REC.NETWORK_CODE ;
        V_ENV_INFO_TRACE.USER_MESSAGE :=  NULL;
        PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
        RETURN( DECLARATION_CST.ERROR );
END PUT_ON_STG_ACCOUNT_DELINQ_CR;
--------------------------------------------------------------------------------------------------------------------------------------
FUNCTION        PUT_ON_STG_ACCOUNT_DELINQ_CC  (P_STG_STG_ACCT_DELINQ_CC_REC  IN           STG_ACCOUNT_DELINQUENCIES_CC%ROWTYPE)
                                                RETURN PLS_INTEGER IS
V_ENV_INFO_TRACE              GLOBAL_VARS.ENV_INFO_TRACE_TYPE:= NULL;
BEGIN
        V_ENV_INFO_TRACE.USER_NAME      :=  GLOBAL_VARS.USER_NAME;
        V_ENV_INFO_TRACE.MODULE_CODE    :=  GLOBAL_VARS.ML_STATISTICS;
        V_ENV_INFO_TRACE.PACKAGE_NAME   :=  $$PLSQL_UNIT;
        V_ENV_INFO_TRACE.FUNCTION_NAME  :=  'PUT_ON_STG_ACCOUNT_DELINQ_CC';
        V_ENV_INFO_TRACE.LANG           :=  GLOBAL_VARS.LANG;

        INSERT INTO STG_ACCOUNT_DELINQUENCIES_CC VALUES  P_STG_STG_ACCT_DELINQ_CC_REC;
        RETURN(DECLARATION_CST.OK);
EXCEPTION
WHEN    OTHERS  THEN
        V_ENV_INFO_TRACE.ERROR_CODE   := 'PWC-00016';
        V_ENV_INFO_TRACE.PARAM1       := 'STG_AUTH_STATISTICS_CC';
        V_ENV_INFO_TRACE.PARAM2       := 'STATISTICS_PERIOD = ' ||  P_STG_STG_ACCT_DELINQ_CC_REC.STATISTICS_PERIOD ||
                                        ', BANK_CODE = ' || P_STG_STG_ACCT_DELINQ_CC_REC.BANK_CODE ||
                                        ', NETWORK_CODE = ' || P_STG_STG_ACCT_DELINQ_CC_REC.NETWORK_CODE ;
        V_ENV_INFO_TRACE.USER_MESSAGE :=  NULL;
        PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
        RETURN( DECLARATION_CST.ERROR );
END PUT_ON_STG_ACCOUNT_DELINQ_CC;
--------------------------------------------------------------------------------------------------------------------------------------
/*FUNCTION   PREP_STG_ACCOUNT_DELINQ_CR   (   P_BUSINESS_DATE                     IN          DATE,
                                            P_CUTOFF_SEQUENCE                   IN          CUTOFF_FOLLOW_UP.CUTOFF_SEQUENCE%TYPE,
                                            P_LANGUAGE_1                        IN          PWC_BANK_REPORT_PARAMETERS.LANGUAGE_1%TYPE,
                                            P_LANGUAGE_2                        IN          PWC_BANK_REPORT_PARAMETERS.LANGUAGE_2%TYPE,
                                            P_LANGUAGE_3                        IN          PWC_BANK_REPORT_PARAMETERS.LANGUAGE_3%TYPE,
                                            P_BANK_RECORD                       IN          BANK%ROWTYPE,
                                            P_NETWORK_RECORD                    IN          NETWORK%ROWTYPE,
                                            P_START_DATE                        IN          DATE,
                                            P_END_DATE                          IN          DATE)
                                            RETURN PLS_INTEGER IS

RETURN_STATUS                           PLS_INTEGER;
V_START_DATE                            DATE;
V_COMPTEUR                              PLS_INTEGER;
V_STG_ACCOUNT_DELINQ_CR_REC             STG_ACCOUNT_DELINQUENCIES_CR%ROWTYPE;
V_AVAILABLE_BALANCE                     SHADOW_ACCOUNT.CLOSING_BALANCE_PURCHASE%TYPE;
V_GLOBAL_BALANCE                        SHADOW_ACCOUNT.CLOSING_BALANCE_PURCHASE%TYPE;
V_GLOBAL_PRINCIPAL_BALANCE              SHADOW_ACCOUNT.CLOSING_BALANCE_PURCHASE%TYPE;
V_CR_TERM_RECORD                        CR_TERM%ROWTYPE;
V_DAYS_AFTER_DUEDATE                    NUMBER;
V_CURRENCY_TABLE_RECORD                 CURRENCY_TABLE%ROWTYPE ;
V_TOT_CREDIT_LIMIT_LC                   SHADOW_ACCOUNT.CREDIT_LIMIT%TYPE;
V_CREDIT_LIMIT_LC                       SHADOW_ACCOUNT.CREDIT_LIMIT%TYPE;
V_CONVERSION_DATE                       CONVERSION_RATE.RATE_DATE%TYPE;
V_RATE_ISO_STR_FORMAT                   VARCHAR2(9);
V_MARKUP_AMOUNT                         TRANSACTION_HIST.CONVERSION_FEES%TYPE;
V_RATE_DATE                             DATE;

CURSOR  CUR_SHADOW_ACCOUNT     IS
SELECT  *
FROM SHADOW_ACCOUNT
WHERE BANK_CODE = P_BANK_RECORD.BANK_CODE
AND   PRIMARY_CARD_PRODUCT_CODE IN (SELECT PRODUCT_CODE
                                    FROM   CARD_PRODUCT
                                    WHERE  PRODUCT_TYPE = GLOBAL_VARS.CREDIT_CARD_TYPE
                                    AND    NETWORK_CODE = P_NETWORK_RECORD.NETWORK_CODE
                                    AND    BANK_CODE    = P_BANK_RECORD.BANK_CODE)
AND  AGREEMENT_DATE <= P_END_DATE     ; --SAB30062015

BEGIN
          V_ENV_INFO_TRACE.USER_NAME      :=  GLOBAL_VARS.USER_NAME;
          V_ENV_INFO_TRACE.MODULE_CODE    :=  GLOBAL_VARS.ML_STATISTICS;
          V_ENV_INFO_TRACE.LANG           :=  GLOBAL_VARS.LANG;
          V_ENV_INFO_TRACE.PACKAGE_NAME   :=  $$PLSQL_UNIT;
          V_ENV_INFO_TRACE.FUNCTION_NAME  :=  'PREP_STG_ACCOUNT_DELINQ_CR';

    RETURN_STATUS   :=  PCRD_GET_PARAM_GENERAL_ROWS.GET_CURRENCY_TABLE  (   P_BANK_RECORD.BANK_CURRENCY_CODE,
                                                                            V_CURRENCY_TABLE_RECORD,
                                                                            TRUE);
    IF RETURN_STATUS <> DECLARATION_CST.OK
    THEN
            V_ENV_INFO_TRACE.USER_MESSAGE := ':ERROR RETURNED BY PCRD_GET_PARAM_GENERAL_ROWS.GET_CURRENCY_TABLE';
            PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
            RETURN(RETURN_STATUS);
    END IF;
    V_STG_ACCOUNT_DELINQ_CR_REC                                :=  NULL;
    V_STG_ACCOUNT_DELINQ_CR_REC.BUSINESS_DATE                  :=  P_BUSINESS_DATE;
    V_STG_ACCOUNT_DELINQ_CR_REC.BANK_CODE                      :=  P_BANK_RECORD.BANK_CODE;
    V_STG_ACCOUNT_DELINQ_CR_REC.BANK_NAME                      :=  P_BANK_RECORD.BANK_NAME;
    V_STG_ACCOUNT_DELINQ_CR_REC.NETWORK_CODE                   :=  P_NETWORK_RECORD.NETWORK_CODE;
    V_STG_ACCOUNT_DELINQ_CR_REC.NETWORK_NAME                   :=  P_NETWORK_RECORD.NETWORK_LABEL;
    V_STG_ACCOUNT_DELINQ_CR_REC.CUTOFF_ID                      :=  P_CUTOFF_SEQUENCE;
    V_STG_ACCOUNT_DELINQ_CR_REC.LANGUAGE_1                     :=  P_LANGUAGE_1;
    V_STG_ACCOUNT_DELINQ_CR_REC.LANGUAGE_2                     :=  P_LANGUAGE_2;
    V_STG_ACCOUNT_DELINQ_CR_REC.LANGUAGE_3                     :=  P_LANGUAGE_3;
    V_STG_ACCOUNT_DELINQ_CR_REC.STATISTICS_PERIOD              :=  P_START_DATE;
    V_STG_ACCOUNT_DELINQ_CR_REC.CURRENCY_CODE                  :=  V_CURRENCY_TABLE_RECORD.CURRENCY_CODE;
    V_STG_ACCOUNT_DELINQ_CR_REC.CURRENCY_EXPONENT              :=  V_CURRENCY_TABLE_RECORD.CURRENCY_EXPONENT;
    V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQ_ACC_LESS_30_DAYS    :=  0.;
    V_STG_ACCOUNT_DELINQ_CR_REC.VAL_DELINQ_ACC_LESS_30_DAYS    :=  0.;
    V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQ_ACC_OVER_120_DAYS   :=  0.;
    V_STG_ACCOUNT_DELINQ_CR_REC.VAL_DELINQ_ACC_OVER_120_DAYS   :=  0.;
    V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQUENT_ACC_30_60_DAYS  :=  0.;
    V_STG_ACCOUNT_DELINQ_CR_REC.VAL_DELINQUENT_ACC_30_60_DAYS  :=  0.;
    V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQUENT_ACC_60_90_DAYS  :=  0.;
    V_STG_ACCOUNT_DELINQ_CR_REC.VAL_DELINQUENT_ACC_60_90_DAYS  :=  0.;
    V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQUENT_ACC_90_120_DAYS :=  0.;
    V_STG_ACCOUNT_DELINQ_CR_REC.VAL_DELINQUENT_ACC_90_120_DAYS :=  0.;
    V_STG_ACCOUNT_DELINQ_CR_REC.TOTAL_NBR_DELINQUENT_ACCOUNTS  :=  0.;
    V_STG_ACCOUNT_DELINQ_CR_REC.TOTAL_VAL_DELINQUENT_ACCOUNTS  :=  0.;
    V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQ_ACC_WITH_GRACE_PER  :=  0.;
    V_STG_ACCOUNT_DELINQ_CR_REC.VAL_DELINQ_ACC_WITH_GRACE_PER  :=  0.;
    FOR V_CUR_SHADOW_ACCOUNT   IN  CUR_SHADOW_ACCOUNT
    LOOP
        IF  V_CUR_SHADOW_ACCOUNT.OWNER_CATEGORY IN (    GLOBAL_VARS.OWNER_CATEGORY_CORPORATE,
                                                        GLOBAL_VARS.OWNER_CATEGORY_CORPORATE_TRAV,
                                                        GLOBAL_VARS.OWNER_CATEGORY_CORPORATE_INT,
                                                        GLOBAL_VARS.OWNER_CATEGORY_COMPANY )
        THEN
            RETURN_STATUS := PCRD_REVOLVING_PROC_TOOLS_7.GET_AVAIALABLE_BALANCE    (    NULL,
                                                                                        V_CUR_SHADOW_ACCOUNT,
                                                                                        V_AVAILABLE_BALANCE,
                                                                                        V_GLOBAL_BALANCE,
                                                                                        V_GLOBAL_PRINCIPAL_BALANCE);
            IF RETURN_STATUS <> 0
            THEN
                V_ENV_INFO_TRACE.USER_MESSAGE   := 'ERROR RETURNED BY PCRD_REVOLVING_PROC_TOOLS_7.GET_AVAIALABLE_BALANCE'
                                                    || ', SHADOW ACCOUNT: '  || V_CUR_SHADOW_ACCOUNT.SHADOW_ACCOUNT_NBR;
                PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
                RETURN  RETURN_STATUS;
            END IF;
        ELSE
            RETURN_STATUS := PCRD_REVOLVING_PROC_TOOLS_3.GET_BALANCE (  V_CUR_SHADOW_ACCOUNT,
                                                                        V_GLOBAL_BALANCE);

            IF  RETURN_STATUS <> DECLARATION_CST.OK
            THEN
                V_ENV_INFO_TRACE.USER_MESSAGE :=  'ERROR RETURNED BY PCRD_REVOLVING_PROC_TOOLS_3.GET_BALANCE: SA = '
                                              ||V_CUR_SHADOW_ACCOUNT.SHADOW_ACCOUNT_NBR;
                PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
                RETURN (RETURN_STATUS);
            END IF;
        END IF;

        BEGIN
            SELECT   *
            INTO    V_CR_TERM_RECORD
                FROM ( SELECT  *
                        FROM   CR_TERM
                        WHERE  SHADOW_ACCOUNT_NBR = V_CUR_SHADOW_ACCOUNT.SHADOW_ACCOUNT_NBR
                        AND     MINIMUM_DUE > NVL(TOTAL_AMOUNT_POSTED,0.0)
                        ORDER BY STATEMENT_DATE )
                  WHERE ROWNUM = 1;
                  V_DAYS_AFTER_DUEDATE  :=  LAST_DAY(P_END_DATE) - V_CR_TERM_RECORD.TERM_OVERDUE_DATE;--V_CR_TERM_RECORD.TERM_DUE_DATE;SAB17042015
        EXCEPTION
        WHEN NO_DATA_FOUND THEN
            V_DAYS_AFTER_DUEDATE := -1;
        WHEN OTHERS THEN
            V_ENV_INFO_TRACE.USER_MESSAGE   := 'SHADOW ACCOUNT : '||V_CUR_SHADOW_ACCOUNT.SHADOW_ACCOUNT_NBR;
            PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
            RETURN (DECLARATION_CST.ERROR);
        END;

        IF (NVL(V_GLOBAL_BALANCE,0.0) <> 0.0) --SAB09102015
        THEN
            IF  V_DAYS_AFTER_DUEDATE < 0
            THEN
                V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQ_ACC_WITH_GRACE_PER    :=  NVL(V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQ_ACC_WITH_GRACE_PER,0) + 1;
                V_STG_ACCOUNT_DELINQ_CR_REC.VAL_DELINQ_ACC_WITH_GRACE_PER    :=  NVL(V_STG_ACCOUNT_DELINQ_CR_REC.VAL_DELINQ_ACC_WITH_GRACE_PER,0.0)
                                                                                    + NVL(V_GLOBAL_BALANCE,0.0);
            ELSIF V_DAYS_AFTER_DUEDATE < 30
            THEN
                V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQ_ACC_LESS_30_DAYS      :=  NVL(V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQ_ACC_LESS_30_DAYS,0) + 1;
                V_STG_ACCOUNT_DELINQ_CR_REC.VAL_DELINQ_ACC_LESS_30_DAYS      :=  NVL(V_STG_ACCOUNT_DELINQ_CR_REC.VAL_DELINQ_ACC_LESS_30_DAYS,0.0)
                                                                                    + NVL(V_GLOBAL_BALANCE,0.0);
            ELSIF   V_DAYS_AFTER_DUEDATE < 60
            THEN
                V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQUENT_ACC_30_60_DAYS    :=  NVL(V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQUENT_ACC_30_60_DAYS,0) + 1;
                V_STG_ACCOUNT_DELINQ_CR_REC.VAL_DELINQUENT_ACC_30_60_DAYS    :=  NVL(V_STG_ACCOUNT_DELINQ_CR_REC.VAL_DELINQUENT_ACC_30_60_DAYS,0.0)
                                                                                    + NVL(V_GLOBAL_BALANCE,0.0);
            ELSIF   V_DAYS_AFTER_DUEDATE < 90
            THEN
                V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQUENT_ACC_60_90_DAYS    :=  NVL(V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQUENT_ACC_60_90_DAYS,0) + 1;
                V_STG_ACCOUNT_DELINQ_CR_REC.VAL_DELINQUENT_ACC_60_90_DAYS    :=  NVL(V_STG_ACCOUNT_DELINQ_CR_REC.VAL_DELINQUENT_ACC_60_90_DAYS,0.0)
                                                                                    + NVL(V_GLOBAL_BALANCE,0.0);
            ELSIF   V_DAYS_AFTER_DUEDATE < 120
            THEN
                V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQUENT_ACC_90_120_DAYS   :=  NVL(V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQUENT_ACC_90_120_DAYS,0) + 1;
                V_STG_ACCOUNT_DELINQ_CR_REC.VAL_DELINQUENT_ACC_90_120_DAYS   :=  NVL(V_STG_ACCOUNT_DELINQ_CR_REC.VAL_DELINQUENT_ACC_90_120_DAYS,0.0)
                                                                                    + NVL(V_GLOBAL_BALANCE,0.0);
            ELSE
                V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQ_ACC_OVER_120_DAYS     :=  NVL(V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQ_ACC_OVER_120_DAYS,0) + 1;
                V_STG_ACCOUNT_DELINQ_CR_REC.VAL_DELINQ_ACC_OVER_120_DAYS     :=  NVL(V_STG_ACCOUNT_DELINQ_CR_REC.VAL_DELINQ_ACC_OVER_120_DAYS,0.0)
                                                                                    + NVL(V_GLOBAL_BALANCE,0.0);
            END IF;
        END IF;
        IF (V_CUR_SHADOW_ACCOUNT.ADMINISTRATIVE_STATUS IN ('0','1','2','3','4'))
        THEN
            RETURN_STATUS       :=  PCRD_CAI_GENERAL_TOOLS_1.CROSS_RATES_CONVERSION(GLOBAL_VARS.NETWORK_VISA,
                                                                                V_CUR_SHADOW_ACCOUNT.CURRENCY_CODE,
                                                                                P_BANK_RECORD.BANK_CURRENCY_CODE,
                                                                                GLOBAL_VARS.RATE_ORIGIN_VISA,
                                                                                GLOBAL_VARS.CONV_MODE_BAS_BUYING,
                                                                                V_CUR_SHADOW_ACCOUNT.CREDIT_LIMIT,
                                                                                V_RATE_DATE,
                                                                                V_RATE_ISO_STR_FORMAT,
                                                                                V_CREDIT_LIMIT_LC );

            IF RETURN_STATUS = DECLARATION_CST.ERROR
            THEN
                    V_ENV_INFO_TRACE.USER_MESSAGE := 'ERROR RETURNED BY PCRD_CAI_GENERAL_TOOLS_1.CROSS_RATES_CONVERSION:'         ||
                                         'FROM CURRENCY :   '   ||  V_CUR_SHADOW_ACCOUNT.CURRENCY_CODE    ||
                                         'TO CURRENCY   :   '   ||  P_BANK_RECORD.BANK_CURRENCY_CODE;
                    PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
                    RETURN DECLARATION_CST.ERROR;
            ELSIF RETURN_STATUS = DECLARATION_CST.NOK
            THEN
                RETURN_STATUS       :=  PCRD_GENERAL_TOOLS.CONVERT_CURRENCY_AMOUNT( P_BANK_RECORD.BANK_CODE,
                                                                                    P_BANK_RECORD.BANK_CURRENCY_CODE,
                                                                                    V_CUR_SHADOW_ACCOUNT.CURRENCY_CODE,
                                                                                    GLOBAL_VARS.RATE_ORIGIN_CENTER,
                                                                                    GLOBAL_VARS.CONV_MODE_BAS_BUYING,
                                                                                    P_BANK_RECORD.BANK_CURRENCY_CODE,
                                                                                    V_CUR_SHADOW_ACCOUNT.CREDIT_LIMIT,
                                                                                    P_BUSINESS_DATE,
                                                                                    NULL,
                                                                                    V_CREDIT_LIMIT_LC,
                                                                                    V_CONVERSION_DATE,
                                                                                    V_RATE_ISO_STR_FORMAT,
                                                                                    V_MARKUP_AMOUNT,
                                                                                    FALSE,
                                                                                    NULL,
                                                                                    FALSE,
                                                                                    NULL );
                IF RETURN_STATUS <> DECLARATION_CST.OK
                THEN
                    V_ENV_INFO_TRACE.USER_MESSAGE    := 'ERROR CALLING PCRD_GENERAL_TOOLS.CONVERT_CURRENCY_AMOUNT '
                                                        ||' FROM CURRENCY : '           ||  V_CUR_SHADOW_ACCOUNT.CURRENCY_CODE
                                                        ||'   TO  CURRENCY    :   '     ||  P_BANK_RECORD.BANK_CURRENCY_CODE;
                    PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
                    RETURN(RETURN_STATUS);
                END IF;
            END IF;
            V_TOT_CREDIT_LIMIT_LC :=  NVL(V_TOT_CREDIT_LIMIT_LC,0) + V_CREDIT_LIMIT_LC;
        END IF;
    END LOOP;

    V_STG_ACCOUNT_DELINQ_CR_REC.TOTAL_NBR_DELINQUENT_ACCOUNTS            :=  NVL(V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQ_ACC_WITH_GRACE_PER,0)
                                                                            +   NVL(V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQ_ACC_LESS_30_DAYS,0)
                                                                            +   NVL(V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQUENT_ACC_30_60_DAYS,0)
                                                                            +   NVL(V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQUENT_ACC_60_90_DAYS,0)
                                                                            +   NVL(V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQUENT_ACC_90_120_DAYS,0)
                                                                            +   NVL(V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQ_ACC_OVER_120_DAYS,0) ;

    V_STG_ACCOUNT_DELINQ_CR_REC.TOTAL_VAL_DELINQUENT_ACCOUNTS            :=  NVL(V_STG_ACCOUNT_DELINQ_CR_REC.VAL_DELINQ_ACC_WITH_GRACE_PER,0)
                                                                            +   NVL(V_STG_ACCOUNT_DELINQ_CR_REC.VAL_DELINQ_ACC_LESS_30_DAYS,0)
                                                                            +   NVL(V_STG_ACCOUNT_DELINQ_CR_REC.VAL_DELINQUENT_ACC_30_60_DAYS,0)
                                                                            +   NVL(V_STG_ACCOUNT_DELINQ_CR_REC.VAL_DELINQUENT_ACC_60_90_DAYS,0)
                                                                            +   NVL(V_STG_ACCOUNT_DELINQ_CR_REC.VAL_DELINQUENT_ACC_90_120_DAYS,0)
                                                                            +   NVL(V_STG_ACCOUNT_DELINQ_CR_REC.VAL_DELINQ_ACC_OVER_120_DAYS,0) ;

    RETURN_STATUS   :=  PCRD_STA_REPORTING_TOOLS_1_V3.PUT_ON_STG_ACCOUNT_DELINQ_CR(V_STG_ACCOUNT_DELINQ_CR_REC);
    IF (RETURN_STATUS <> DECLARATION_CST.OK)
    THEN
        V_ENV_INFO_TRACE.USER_MESSAGE := 'ERROR RETURNED BY PCRD_STA_REPORTING_TOOLS_1_V3.PUT_ON_STG_ACCOUNT_DELINQ_CR, STATISTICS_PERIOD = ' ||  V_STG_ACCOUNT_DELINQ_CR_REC.STATISTICS_PERIOD ||
                            ', BANK_CODE = ' || V_STG_ACCOUNT_DELINQ_CR_REC.BANK_CODE ||
                            ', NETWORK_CODE = ' || V_STG_ACCOUNT_DELINQ_CR_REC.NETWORK_CODE ;
        PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
        RETURN (DECLARATION_CST.ERROR);
    END IF;
    BEGIN
        UPDATE STG_CARD_STATISTICS_CR
        SET TOT_OF_CREDIT_LIMIT_LC =  NVL(V_TOT_CREDIT_LIMIT_LC,0.)
        WHERE   BANK_CODE = P_BANK_RECORD.BANK_CODE
        AND     NETWORK_CODE = P_NETWORK_RECORD.NETWORK_CODE
        AND     STATISTICS_PERIOD             =  ADD_MONTHS(P_START_DATE, 2)
        AND     ROWNUM                        = 1;
        EXCEPTION WHEN NO_DATA_FOUND
        THEN
            NULL;
        WHEN OTHERS
        THEN
            V_ENV_INFO_TRACE.USER_MESSAGE   := 'ERROR WHEN UPDATE STG_CARD_STATISTICS_CR';
            PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
            RETURN DECLARATION_CST.ERROR;
    END;
    RETURN(DECLARATION_CST.OK);
EXCEPTION WHEN OTHERS
THEN
    V_ENV_INFO_TRACE.USER_MESSAGE   :=  NULL;
    PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
    RETURN(DECLARATION_CST.NOK);
END PREP_STG_ACCOUNT_DELINQ_CR;*/
--------------------------------------------------------------------------------------------------------------------
FUNCTION        PUT_ON_STG_BRCH_CARDS_BLOCKED  (P_STG_BRANCH_CARDS_BLOCKED_REC  IN           STG_BRANCH_CARDS_BLOCKED%ROWTYPE)
                                                RETURN PLS_INTEGER IS
V_ENV_INFO_TRACE              GLOBAL_VARS.ENV_INFO_TRACE_TYPE:= NULL;
BEGIN
        V_ENV_INFO_TRACE.USER_NAME      :=  GLOBAL_VARS.USER_NAME;
        V_ENV_INFO_TRACE.MODULE_CODE    :=  GLOBAL_VARS.ML_STATISTICS;
        V_ENV_INFO_TRACE.PACKAGE_NAME   :=  $$PLSQL_UNIT;
        V_ENV_INFO_TRACE.FUNCTION_NAME  :=  'PUT_ON_STG_BRCH_CARDS_BLOCKED';
        V_ENV_INFO_TRACE.LANG           :=  GLOBAL_VARS.LANG;

        INSERT INTO STG_BRANCH_CARDS_BLOCKED VALUES  P_STG_BRANCH_CARDS_BLOCKED_REC;
        RETURN(DECLARATION_CST.OK);
EXCEPTION
WHEN    OTHERS  THEN
        V_ENV_INFO_TRACE.ERROR_CODE   := 'PWC-00016';
        V_ENV_INFO_TRACE.PARAM1       := 'STG_BRANCH_CARDS_BLOCKED';
        V_ENV_INFO_TRACE.PARAM2       := 'STATISTICS_PERIOD = ' ||  P_STG_BRANCH_CARDS_BLOCKED_REC.STATISTICS_PERIOD ||
                                        ', BANK_CODE = ' || P_STG_BRANCH_CARDS_BLOCKED_REC.BANK_CODE ||
                                        ', BRANCH_CODE = ' || P_STG_BRANCH_CARDS_BLOCKED_REC.BRANCH_CODE ;
        V_ENV_INFO_TRACE.USER_MESSAGE :=  NULL;
        PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
        RETURN( DECLARATION_CST.ERROR );
END PUT_ON_STG_BRCH_CARDS_BLOCKED;
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
FUNCTION    SAVE_STG_BRCH_CARDS_BLOCKED               RETURN PLS_INTEGER IS

    RETURN_STATUS               PLS_INTEGER;
    V_ENV_INFO_TRACE            GLOBAL_VARS.ENV_INFO_TRACE_TYPE     :=      NULL;
BEGIN
    V_ENV_INFO_TRACE.USER_NAME      :=  GLOBAL_VARS.USER_NAME;
    V_ENV_INFO_TRACE.MODULE_CODE    :=  GLOBAL_VARS.ML_ATM;
    V_ENV_INFO_TRACE.PACKAGE_NAME   :=  $$PLSQL_UNIT;
    V_ENV_INFO_TRACE.FUNCTION_NAME  :=  'SAVE_STG_BRCH_CARDS_BLOCKED';
    V_ENV_INFO_TRACE.LANG           :=  GLOBAL_VARS.LANG;
    BEGIN
        INSERT INTO STG_BRANCH_CARDS_BLOCKED_HIST SELECT * FROM STG_BRANCH_CARDS_BLOCKED;
        EXCEPTION
            WHEN OTHERS THEN
                V_ENV_INFO_TRACE.ERROR_CODE   := 'PWC-00016';
                V_ENV_INFO_TRACE.PARAM1       := 'STG_BRANCH_CARDS_BLOCKED_HIST';
                V_ENV_INFO_TRACE.USER_MESSAGE :=  NULL;
                PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
                RETURN(DECLARATION_CST.ERROR);
    END;
    BEGIN
        DELETE FROM STG_BRANCH_CARDS_BLOCKED;
        EXCEPTION
            WHEN OTHERS THEN
                PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
                RETURN DECLARATION_CST.ERROR;
    END;
    RETURN(DECLARATION_CST.OK);
    EXCEPTION
    WHEN OTHERS THEN
        V_ENV_INFO_TRACE.USER_MESSAGE :=  NULL;
        PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE, $$PLSQL_LINE);
        RETURN(DECLARATION_CST.ERROR);
END SAVE_STG_BRCH_CARDS_BLOCKED;
--------------------------------------------------------------------------------------------------------------------------------------
FUNCTION        PUT_ON_STG_ACQUIR_STAT_MER  (P_STG_ACQUIR_STAT_MER_REC          IN           STG_ACQUIR_STAT_MER%ROWTYPE)
                                             RETURN PLS_INTEGER IS
V_ENV_INFO_TRACE              GLOBAL_VARS.ENV_INFO_TRACE_TYPE:= NULL;
BEGIN
        V_ENV_INFO_TRACE.USER_NAME      :=  GLOBAL_VARS.USER_NAME;
        V_ENV_INFO_TRACE.MODULE_CODE    :=  GLOBAL_VARS.ML_STATISTICS;
        V_ENV_INFO_TRACE.PACKAGE_NAME   :=  $$PLSQL_UNIT;
        V_ENV_INFO_TRACE.FUNCTION_NAME  :=  'PUT_ON_STG_ACQUIR_STAT_MER';
        V_ENV_INFO_TRACE.LANG           :=  GLOBAL_VARS.LANG;

        INSERT INTO STG_ACQUIR_STAT_MER VALUES  P_STG_ACQUIR_STAT_MER_REC;
        RETURN(DECLARATION_CST.OK);
EXCEPTION
WHEN    OTHERS  THEN
        V_ENV_INFO_TRACE.ERROR_CODE   := 'PWC-00016';
        V_ENV_INFO_TRACE.PARAM1       := 'STG_ACQUIR_STAT_MER';
        V_ENV_INFO_TRACE.PARAM2       := 'STATISTICS_PERIOD = ' ||  P_STG_ACQUIR_STAT_MER_REC.STAT_MONTH ||
                                        ', BANK_CODE = ' || P_STG_ACQUIR_STAT_MER_REC.BANK_CODE ||
                                        ', MERCHANT_NUMBER = ' || P_STG_ACQUIR_STAT_MER_REC.MERCHANT_NUMBER  ;
        V_ENV_INFO_TRACE.USER_MESSAGE :=  NULL;
        PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
        RETURN( DECLARATION_CST.ERROR );
END PUT_ON_STG_ACQUIR_STAT_MER;
--------------------------------------------------------------------------------------------------------------------------------------
--START SAB27102015
/*FUNCTION   PREP_STG_ACCOUNT_DELINQ_DWH_CR(  P_BUSINESS_DATE                     IN          DATE)
                                            RETURN PLS_INTEGER IS

RETURN_STATUS                           PLS_INTEGER;
V_STG_ACCOUNT_DELINQ_CR_REC             STG_ACCOUNT_DELINQUENCIES_CR%ROWTYPE;
V_NETWORK_RECORD                        NETWORK%ROWTYPE;
V_BANK_RECORD                           BANK%ROWTYPE;
V_CURRENCY_TABLE_RECORD                 CURRENCY_TABLE%ROWTYPE;
V_OLD_BANK_CODE                         BANK.BANK_CODE%TYPE;

CURSOR CUR_DWH_ACC_DAILY_SUMMARY IS
SELECT   A.BANK_CODE, DPD_CLASSIFICATION_CODE,
         SUM (OUTSTANDING_AMOUNT) OUTSTANDING_AMOUNT, SUM(CREDIT_LIMIT_APPROVED_AMOUNT) CREDIT_LIMIT,COUNT(1) NBR_OF_ACCOUNT
FROM     DWH_ACC_DAILY_SUMMARY A, CARD_PRODUCT B
WHERE    A.PRODUCT_CODE = B.PRODUCT_CODE
AND      A.BANK_CODE = B.BANK_CODE
AND      PRODUCT_TYPE = GLOBAL_VARS.CREDIT_CARD_TYPE
AND      NETWORK_CODE = GLOBAL_VARS.NETWORK_VISA
GROUP BY A.BANK_CODE,
         DPD_CLASSIFICATION_CODE
ORDER BY A.BANK_CODE;
BEGIN
    IF (TRUNC(P_BUSINESS_DATE) = LAST_DAY(TO_DATE(TO_CHAR(P_BUSINESS_DATE,'DD')||'/'||TO_CHAR(P_BUSINESS_DATE,'MM')||'/'||TO_CHAR(P_BUSINESS_DATE,'YYYY'),'DD/MM/YYYY')))
    THEN
        DELETE FROM  STG_ACCOUNT_DELINQUENCIES_CR
        WHERE  STATISTICS_PERIOD  = TRUNC(P_BUSINESS_DATE,'MONTH');

        V_STG_ACCOUNT_DELINQ_CR_REC.BUSINESS_DATE                  := TRUNC(P_BUSINESS_DATE);
        V_STG_ACCOUNT_DELINQ_CR_REC.STATISTICS_PERIOD              := TRUNC(P_BUSINESS_DATE,'MONTH');
        V_STG_ACCOUNT_DELINQ_CR_REC.NETWORK_CODE                   := GLOBAL_VARS.NETWORK_VISA;
        RETURN_STATUS   :=  PCRD_GET_PARAM_GENERAL_ROWS.GET_NETWORK  ( V_STG_ACCOUNT_DELINQ_CR_REC.NETWORK_CODE,
                                                                        V_NETWORK_RECORD,
                                                                        TRUE);
        IF RETURN_STATUS <> DECLARATION_CST.OK
        THEN
            V_ENV_INFO_TRACE.USER_MESSAGE := ':ERROR RETURNED BY PCRD_GET_PARAM_GENERAL_ROWS.GET_NETWORK '
                                            || 'NETWORK CODE :' ||V_STG_ACCOUNT_DELINQ_CR_REC.NETWORK_CODE;
            PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
            ROLLBACK;
            RETURN (RETURN_STATUS);
        END IF;
        V_STG_ACCOUNT_DELINQ_CR_REC.NETWORK_NAME                    := V_NETWORK_RECORD.NETWORK_LABEL;
        V_STG_ACCOUNT_DELINQ_CR_REC.BANK_CODE                       := 'XXXXXX';
        FOR V_CUR_DWH_ACC_DAILY_SUMMARY   IN  CUR_DWH_ACC_DAILY_SUMMARY
        LOOP
            IF (V_STG_ACCOUNT_DELINQ_CR_REC.BANK_CODE <> V_CUR_DWH_ACC_DAILY_SUMMARY.BANK_CODE)
            THEN
                IF (V_STG_ACCOUNT_DELINQ_CR_REC.BANK_CODE <> 'XXXXXX')
                THEN
                    RETURN_STATUS   :=  PCRD_STA_REPORTING_TOOLS_1_V3.PUT_ON_STG_ACCOUNT_DELINQ_CR(V_STG_ACCOUNT_DELINQ_CR_REC);
                    IF (RETURN_STATUS <> DECLARATION_CST.OK)
                    THEN
                        V_ENV_INFO_TRACE.USER_MESSAGE := 'ERROR RETURNED BY PCRD_STA_REPORTING_TOOLS_1_V3.PUT_ON_STG_ACCOUNT_DELINQ_CR, STATISTICS_PERIOD = ' ||  V_STG_ACCOUNT_DELINQ_CR_REC.STATISTICS_PERIOD ||
                                            ', BANK_CODE = ' || V_STG_ACCOUNT_DELINQ_CR_REC.BANK_CODE ||
                                            ', STATISTICS_PERIOD = ' || V_STG_ACCOUNT_DELINQ_CR_REC.STATISTICS_PERIOD ||
                                            ', NETWORK_CODE = ' || V_STG_ACCOUNT_DELINQ_CR_REC.NETWORK_CODE ;
                        PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
                        RETURN (DECLARATION_CST.ERROR);
                    END IF;
                END IF;
                V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQ_ACC_LESS_30_DAYS    :=  0.;
                V_STG_ACCOUNT_DELINQ_CR_REC.VAL_DELINQ_ACC_LESS_30_DAYS    :=  0.;
                V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQ_ACC_OVER_120_DAYS   :=  0.;
                V_STG_ACCOUNT_DELINQ_CR_REC.VAL_DELINQ_ACC_OVER_120_DAYS   :=  0.;
                V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQUENT_ACC_30_60_DAYS  :=  0.;
                V_STG_ACCOUNT_DELINQ_CR_REC.VAL_DELINQUENT_ACC_30_60_DAYS  :=  0.;
                V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQUENT_ACC_60_90_DAYS  :=  0.;
                V_STG_ACCOUNT_DELINQ_CR_REC.VAL_DELINQUENT_ACC_60_90_DAYS  :=  0.;
                V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQUENT_ACC_90_120_DAYS :=  0.;
                V_STG_ACCOUNT_DELINQ_CR_REC.VAL_DELINQUENT_ACC_90_120_DAYS :=  0.;
                V_STG_ACCOUNT_DELINQ_CR_REC.TOTAL_NBR_DELINQUENT_ACCOUNTS  :=  0.;
                V_STG_ACCOUNT_DELINQ_CR_REC.TOTAL_VAL_DELINQUENT_ACCOUNTS  :=  0.;
                V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQ_ACC_WITH_GRACE_PER  :=  0.;
                V_STG_ACCOUNT_DELINQ_CR_REC.VAL_DELINQ_ACC_WITH_GRACE_PER  :=  0.;
                V_STG_ACCOUNT_DELINQ_CR_REC.TOT_OF_CREDIT_LIMIT_LC         :=  0.;
                V_STG_ACCOUNT_DELINQ_CR_REC.TOT_OF_CREDIT_LIMIT_UC         :=  0.;
                V_STG_ACCOUNT_DELINQ_CR_REC.BANK_CODE                      := V_CUR_DWH_ACC_DAILY_SUMMARY.BANK_CODE;
                RETURN_STATUS := PCRD_GET_PARAM_GENERAL_ROWS.GET_BANK ( V_STG_ACCOUNT_DELINQ_CR_REC.BANK_CODE,
                                                                        V_BANK_RECORD,
                                                                        TRUE );
                IF RETURN_STATUS <> DECLARATION_CST.OK
                THEN
                    V_ENV_INFO_TRACE.USER_MESSAGE   :=  'ERROR RETURNED BY PCRD_GET_PARAM_GENERAL_ROWS.GET_BANK'
                                                    ||  ', BANK_CODE : '         || V_STG_ACCOUNT_DELINQ_CR_REC.BANK_CODE;
                    PCRD_GENERAL_TOOLS.PUT_TRACES  (V_ENV_INFO_TRACE, $$PLSQL_LINE  );
                    RETURN  RETURN_STATUS;
                END IF;
                V_STG_ACCOUNT_DELINQ_CR_REC.BANK_NAME               := V_BANK_RECORD.BANK_NAME;
                V_STG_ACCOUNT_DELINQ_CR_REC.CURRENCY_CODE            := V_BANK_RECORD.BANK_CURRENCY_CODE;
                RETURN_STATUS   :=  PCRD_GET_PARAM_GENERAL_ROWS.GET_CURRENCY_TABLE  ( V_STG_ACCOUNT_DELINQ_CR_REC.CURRENCY_CODE,
                                                                                        V_CURRENCY_TABLE_RECORD,
                                                                                        TRUE);
                IF RETURN_STATUS <> DECLARATION_CST.OK
                THEN
                    V_ENV_INFO_TRACE.USER_MESSAGE   :=  ':ERROR RETURNED BY PCRD_GET_PARAM_GENERAL_ROWS.GET_CURRENCY_TABLE'
                                                ||  ', STAT USER CURRENCY : ' || V_STG_ACCOUNT_DELINQ_CR_REC.CURRENCY_CODE;
                    PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
                    RETURN(RETURN_STATUS);
                END IF;
                V_STG_ACCOUNT_DELINQ_CR_REC.CURRENCY_EXPONENT         := V_CURRENCY_TABLE_RECORD.CURRENCY_EXPONENT;
            END IF;
            CASE V_CUR_DWH_ACC_DAILY_SUMMARY.DPD_CLASSIFICATION_CODE
            WHEN '0' THEN V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQ_ACC_LESS_30_DAYS   := V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQ_ACC_LESS_30_DAYS + NVL(V_CUR_DWH_ACC_DAILY_SUMMARY.NBR_OF_ACCOUNT,0);
                          V_STG_ACCOUNT_DELINQ_CR_REC.VAL_DELINQ_ACC_LESS_30_DAYS   := V_STG_ACCOUNT_DELINQ_CR_REC.VAL_DELINQ_ACC_LESS_30_DAYS + NVL(V_CUR_DWH_ACC_DAILY_SUMMARY.OUTSTANDING_AMOUNT,0);
            WHEN '1' THEN V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQUENT_ACC_30_60_DAYS := V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQUENT_ACC_30_60_DAYS + NVL(V_CUR_DWH_ACC_DAILY_SUMMARY.NBR_OF_ACCOUNT,0);
                          V_STG_ACCOUNT_DELINQ_CR_REC.VAL_DELINQUENT_ACC_30_60_DAYS := V_STG_ACCOUNT_DELINQ_CR_REC.VAL_DELINQUENT_ACC_30_60_DAYS + NVL(V_CUR_DWH_ACC_DAILY_SUMMARY.OUTSTANDING_AMOUNT,0);
            WHEN '2' THEN V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQUENT_ACC_60_90_DAYS := V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQUENT_ACC_60_90_DAYS + NVL(V_CUR_DWH_ACC_DAILY_SUMMARY.NBR_OF_ACCOUNT,0);
                          V_STG_ACCOUNT_DELINQ_CR_REC.VAL_DELINQUENT_ACC_60_90_DAYS := V_STG_ACCOUNT_DELINQ_CR_REC.VAL_DELINQUENT_ACC_60_90_DAYS + NVL(V_CUR_DWH_ACC_DAILY_SUMMARY.OUTSTANDING_AMOUNT,0);
            WHEN '3' THEN V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQUENT_ACC_90_120_DAYS:= V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQUENT_ACC_90_120_DAYS + NVL(V_CUR_DWH_ACC_DAILY_SUMMARY.NBR_OF_ACCOUNT,0);
                          V_STG_ACCOUNT_DELINQ_CR_REC.VAL_DELINQUENT_ACC_90_120_DAYS:= V_STG_ACCOUNT_DELINQ_CR_REC.VAL_DELINQUENT_ACC_90_120_DAYS + NVL(V_CUR_DWH_ACC_DAILY_SUMMARY.OUTSTANDING_AMOUNT,0);

            ELSE
                V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQ_ACC_OVER_120_DAYS := V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQ_ACC_OVER_120_DAYS + NVL(V_CUR_DWH_ACC_DAILY_SUMMARY.NBR_OF_ACCOUNT,0);
            END CASE;
            V_STG_ACCOUNT_DELINQ_CR_REC.TOTAL_NBR_DELINQUENT_ACCOUNTS       := V_STG_ACCOUNT_DELINQ_CR_REC.TOTAL_NBR_DELINQUENT_ACCOUNTS + NVL(V_CUR_DWH_ACC_DAILY_SUMMARY.NBR_OF_ACCOUNT,0);
            V_STG_ACCOUNT_DELINQ_CR_REC.TOTAL_VAL_DELINQUENT_ACCOUNTS       := V_STG_ACCOUNT_DELINQ_CR_REC.TOTAL_VAL_DELINQUENT_ACCOUNTS + NVL(V_CUR_DWH_ACC_DAILY_SUMMARY.OUTSTANDING_AMOUNT,0);
            V_STG_ACCOUNT_DELINQ_CR_REC.TOT_OF_CREDIT_LIMIT_LC              := V_STG_ACCOUNT_DELINQ_CR_REC.TOT_OF_CREDIT_LIMIT_LC + NVL(V_CUR_DWH_ACC_DAILY_SUMMARY.CREDIT_LIMIT,0);
        END LOOP;
        IF (V_STG_ACCOUNT_DELINQ_CR_REC.BANK_CODE <> 'XXXXXX')
        THEN
            RETURN_STATUS   :=  PCRD_STA_REPORTING_TOOLS_1_V3.PUT_ON_STG_ACCOUNT_DELINQ_CR(V_STG_ACCOUNT_DELINQ_CR_REC);
            IF (RETURN_STATUS <> DECLARATION_CST.OK)
            THEN
                V_ENV_INFO_TRACE.USER_MESSAGE := 'ERROR RETURNED BY PCRD_STA_REPORTING_TOOLS_1_V3.PUT_ON_STG_ACCOUNT_DELINQ_CR, STATISTICS_PERIOD = ' ||  V_STG_ACCOUNT_DELINQ_CR_REC.STATISTICS_PERIOD ||
                                    ', BANK_CODE = ' || V_STG_ACCOUNT_DELINQ_CR_REC.BANK_CODE ||
                                    ', STATISTICS_PERIOD = ' || V_STG_ACCOUNT_DELINQ_CR_REC.STATISTICS_PERIOD ||
                                    ', NETWORK_CODE = ' || V_STG_ACCOUNT_DELINQ_CR_REC.NETWORK_CODE ;
                PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
                RETURN (DECLARATION_CST.ERROR);
            END IF;
        END IF;
        COMMIT;
    END IF;
    RETURN(DECLARATION_CST.OK);
EXCEPTION WHEN OTHERS
THEN
    V_ENV_INFO_TRACE.USER_MESSAGE   :=  NULL;
    PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
    RETURN(DECLARATION_CST.NOK);
END PREP_STG_ACCOUNT_DELINQ_DWH_CR;*/
--END SAB27102015
--------------------------------------------------------------------------------------------------------------------------------------
FUNCTION        PUT_ON_STG_CARD_STATISTICS_DC  (P_STG_CARD_STATIST_DC_REC  IN           STG_CARD_STATISTICS_DC%ROWTYPE)
                                                RETURN PLS_INTEGER IS
V_ENV_INFO_TRACE              GLOBAL_VARS.ENV_INFO_TRACE_TYPE:= NULL;
BEGIN
        V_ENV_INFO_TRACE.USER_NAME      :=  GLOBAL_VARS.USER_NAME;
        V_ENV_INFO_TRACE.MODULE_CODE    :=  GLOBAL_VARS.ML_STATISTICS;
        V_ENV_INFO_TRACE.PACKAGE_NAME   :=  $$PLSQL_UNIT;
        V_ENV_INFO_TRACE.FUNCTION_NAME  :=  'PUT_ON_STG_CARD_STATISTICS_DC';
        V_ENV_INFO_TRACE.LANG           :=  GLOBAL_VARS.LANG;

        INSERT INTO STG_CARD_STATISTICS_DC VALUES  P_STG_CARD_STATIST_DC_REC;
        RETURN(DECLARATION_CST.OK);
EXCEPTION
WHEN    OTHERS  THEN
        V_ENV_INFO_TRACE.ERROR_CODE   := 'PWC-00016';
        V_ENV_INFO_TRACE.PARAM1       := 'STG_CARD_STATISTICS_CR';
        V_ENV_INFO_TRACE.PARAM2       := 'STATISTICS_PERIOD = ' ||  P_STG_CARD_STATIST_DC_REC.STATISTICS_PERIOD ||
                                        ', BANK_CODE = ' || P_STG_CARD_STATIST_DC_REC.BANK_CODE ||
                                        ', NETWORK_CODE = ' || P_STG_CARD_STATIST_DC_REC.NETWORK_CODE ||
                                        ', CLASS_GROUP = ' || P_STG_CARD_STATIST_DC_REC.CLASS_GROUP ||
                                         ', CLASS = ' || P_STG_CARD_STATIST_DC_REC.CLASS ;
        V_ENV_INFO_TRACE.USER_MESSAGE :=  NULL;
        PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
        RETURN( DECLARATION_CST.ERROR );
END PUT_ON_STG_CARD_STATISTICS_DC;
----------------------------------------------------------------------------------------------------------------------
FUNCTION        PUT_ON_STG_TRANS_STATISTICS_DC  (P_STG_TRANS_STATIST_DC_REC  IN           STG_TRANSACTION_STATISTICS_DC%ROWTYPE)
                                            RETURN PLS_INTEGER IS
V_ENV_INFO_TRACE              GLOBAL_VARS.ENV_INFO_TRACE_TYPE:= NULL;
BEGIN
        V_ENV_INFO_TRACE.USER_NAME      :=  GLOBAL_VARS.USER_NAME;
        V_ENV_INFO_TRACE.MODULE_CODE    :=  GLOBAL_VARS.ML_STATISTICS;
        V_ENV_INFO_TRACE.PACKAGE_NAME   :=  $$PLSQL_UNIT;
        V_ENV_INFO_TRACE.FUNCTION_NAME  :=  'PUT_ON_STG_TRANS_STATISTICS_DC';
        V_ENV_INFO_TRACE.LANG           :=  GLOBAL_VARS.LANG;

        INSERT INTO STG_TRANSACTION_STATISTICS_DC VALUES  P_STG_TRANS_STATIST_DC_REC;
        RETURN(DECLARATION_CST.OK);
EXCEPTION
WHEN    OTHERS  THEN
        V_ENV_INFO_TRACE.ERROR_CODE   := 'PWC-00016';
        V_ENV_INFO_TRACE.PARAM1       := 'STG_TRANSACTION_STATISTICS_DC';
        V_ENV_INFO_TRACE.PARAM2       := 'STATISTICS_PERIOD = ' ||  P_STG_TRANS_STATIST_DC_REC.STATISTICS_PERIOD ||
                                        ', BANK_CODE = ' || P_STG_TRANS_STATIST_DC_REC.BANK_CODE ||
                                        ', NETWORK_CODE = ' || P_STG_TRANS_STATIST_DC_REC.NETWORK_CODE ||
                                        ', CLASS_GROUP = ' || P_STG_TRANS_STATIST_DC_REC.CLASS_GROUP ||
                                         ', CLASS = ' || P_STG_TRANS_STATIST_DC_REC.CLASS ;
        V_ENV_INFO_TRACE.USER_MESSAGE :=  NULL;
        PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
        RETURN( DECLARATION_CST.ERROR );
END PUT_ON_STG_TRANS_STATISTICS_DC;
----------------------------------------------------------------------------------------------------------------------
FUNCTION   PREP_STG_TRANSAC_STATISTICS_DC ( P_BUSINESS_DATE                     IN          DATE,
                                            P_CUTOFF_SEQUENCE                   IN          CUTOFF_FOLLOW_UP.CUTOFF_SEQUENCE%TYPE,
                                            P_LANGUAGE_1                        IN          PWC_BANK_REPORT_PARAMETERS.LANGUAGE_1%TYPE,
                                            P_LANGUAGE_2                        IN          PWC_BANK_REPORT_PARAMETERS.LANGUAGE_2%TYPE,
                                            P_LANGUAGE_3                        IN          PWC_BANK_REPORT_PARAMETERS.LANGUAGE_3%TYPE,
                                            P_BANK_RECORD                       IN          BANK%ROWTYPE,
                                            P_NETWORK_RECORD                    IN          NETWORK%ROWTYPE,
                                            P_CLASS                             IN          CARD_TYPE.CLASS%TYPE,
                                            P_CLASS_GROUP                       IN          CARD_TYPE.CLASS_GROUP%TYPE,
                                            P_START_DATE                        IN          DATE)
                                            RETURN PLS_INTEGER IS
RETURN_STATUS                           PLS_INTEGER;
V_KEY_VAL                               RESSOURCE_BUNDLE.KEY_VAL%TYPE;
V_MULTI_LANG_REPORT_V3                  MULTI_LANG_REPORT_V3;
V_BUNDLE_REPORT_V3                      BUNDLE_REPORT_V3;
V_START_DATE                            DATE;
V_COMPTEUR                              PLS_INTEGER;
V_STG_TRANS_STATISTICS_DC_REC           STG_TRANSACTION_STATISTICS_DC%ROWTYPE;
V_TRANSACTION_AMOUNT_UC                 NUMBER(18,3); --MBH10012018
V_CURRENCY_TABLE_RECORD                 CURRENCY_TABLE%ROWTYPE;

CURSOR CUR_M_STAT_TRANSACTIONS (P_BANK_CODE BANK.BANK_CODE%TYPE,P_NETWORK_CODE  NETWORK.NETWORK_CODE%TYPE , P_CLASS CARD_TYPE.CLASS%TYPE, P_CLASS_GROUP CARD_TYPE.CLASS_GROUP%TYPE, P_START_DATE    DATE) IS
    SELECT   M.*
    FROM     M_STAT_TRANSACTIONS M, CARD_PRODUCT C
    WHERE    STATISTICS_PERIOD          = TRUNC(TO_DATE(P_START_DATE),'MONTH')
    AND      M.ISSUER_BANK_CODE         = P_BANK_CODE
    AND      M.CLASS                    = P_CLASS
    AND      M.CLASS_GROUP              = P_CLASS_GROUP
    AND      M.NETWORK_CODE             = P_NETWORK_CODE
    AND      M.SOURCE_TRX               = 'T'
    AND      M.ISSUER_BANK_CODE         = C.BANK_CODE                   -- TED20160811
    AND      M.CARD_PRODUCT_CODE        = C.PRODUCT_CODE                -- TED20160811
    AND      C.PRODUCT_TYPE             = GLOBAL_VARS.DEBIT_CARD_TYPE;  -- TED20160811
--    AND      CARD_PRODUCT_TYPE         = GLOBAL_VARS.DEBIT_CARD_TYPE; -- TED20160811
BEGIN
      V_ENV_INFO_TRACE.USER_NAME      :=  GLOBAL_VARS.USER_NAME;
      V_ENV_INFO_TRACE.MODULE_CODE    :=  GLOBAL_VARS.ML_STATISTICS;
      V_ENV_INFO_TRACE.LANG           :=  GLOBAL_VARS.LANG;
      V_ENV_INFO_TRACE.PACKAGE_NAME   :=  $$PLSQL_UNIT;
      V_ENV_INFO_TRACE.FUNCTION_NAME  :=  'PREP_STG_TRANSAC_STATISTICS_DC';

                V_START_DATE :=  TRUNC(P_START_DATE);
                V_COMPTEUR := 1;
                LOOP
                    V_STG_TRANS_STATISTICS_DC_REC                                :=  NULL;
                    V_STG_TRANS_STATISTICS_DC_REC.BANK_CODE                      :=  P_BANK_RECORD.BANK_CODE;
                    V_STG_TRANS_STATISTICS_DC_REC.BANK_NAME                      :=  P_BANK_RECORD.BANK_NAME;
                    V_STG_TRANS_STATISTICS_DC_REC.NETWORK_CODE                   :=  P_NETWORK_RECORD.NETWORK_CODE;
                    V_STG_TRANS_STATISTICS_DC_REC.NETWORK_NAME                   :=  P_NETWORK_RECORD.NETWORK_LABEL;
                    V_STG_TRANS_STATISTICS_DC_REC.BUSINESS_DATE                  :=  P_BUSINESS_DATE;
                    V_STG_TRANS_STATISTICS_DC_REC.CUTOFF_ID                      :=  P_CUTOFF_SEQUENCE;
                    V_STG_TRANS_STATISTICS_DC_REC.LANGUAGE_1                     :=  P_LANGUAGE_1;
                    V_STG_TRANS_STATISTICS_DC_REC.LANGUAGE_2                     :=  P_LANGUAGE_2;
                    V_STG_TRANS_STATISTICS_DC_REC.LANGUAGE_3                     :=  P_LANGUAGE_3;
                    V_STG_TRANS_STATISTICS_DC_REC.STATISTICS_PERIOD              :=  V_START_DATE;
                    V_STG_TRANS_STATISTICS_DC_REC.CLASS_GROUP                    :=  P_CLASS_GROUP;
                    V_KEY_VAL := '';
                    CASE    V_STG_TRANS_STATISTICS_DC_REC.CLASS_GROUP
                    WHEN    'IN'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS_GROUP.IN';
                    WHEN    'BU'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS_GROUP.BU';
                    ELSE    V_KEY_VAL := 'NA';
                    END CASE;
                    V_BUNDLE_REPORT_V3 := PCRD_REPORTING_TOOLS_V3.NEW_BUNDLE_REPORT_V3;
                    V_BUNDLE_REPORT_V3.KEY_VAL                      := V_KEY_VAL;
                    V_BUNDLE_REPORT_V3.LANGUAGE_1                   := V_STG_TRANS_STATISTICS_DC_REC.LANGUAGE_1;
                    V_BUNDLE_REPORT_V3.LANGUAGE_2                   := V_STG_TRANS_STATISTICS_DC_REC.LANGUAGE_2;
                    V_BUNDLE_REPORT_V3.LANGUAGE_3                   := V_STG_TRANS_STATISTICS_DC_REC.LANGUAGE_3;
                    RETURN_STATUS   :=  PCRD_REPORTING_TOOLS_V3.GET_REPORT_LABELS   ( V_BUNDLE_REPORT_V3 );
                    IF RETURN_STATUS <> DECLARATION_CST.OK
                    THEN
                        V_ENV_INFO_TRACE.USER_MESSAGE   :=  'ERROR RETURNED BY PCRD_REPORTING_TOOLS_V3.GET_REPORT_LABELS'  ||
                                                            ',BUNDLE : ' || PCRD_REPORTING_TOOLS_V3.BUNDLE ||
                                                            ',KEY_VAL : ' || V_KEY_VAL;
                        PCRD_GENERAL_TOOLS.PUT_TRACES  (V_ENV_INFO_TRACE, $$PLSQL_LINE  );
                        RETURN  ( DECLARATION_CST.ERROR );
                    END IF;
                    V_STG_TRANS_STATISTICS_DC_REC.CLASS_GROUP_DESC_1             :=  V_BUNDLE_REPORT_V3.VALUE_1;
                    V_STG_TRANS_STATISTICS_DC_REC.CLASS_GROUP_DESC_2             :=  V_BUNDLE_REPORT_V3.VALUE_2;
                    V_STG_TRANS_STATISTICS_DC_REC.CLASS_GROUP_DESC_3             :=  V_BUNDLE_REPORT_V3.VALUE_3;
                    V_STG_TRANS_STATISTICS_DC_REC.CLASS                          :=  P_CLASS;
                    V_KEY_VAL := '';
                    CASE    V_STG_TRANS_STATISTICS_DC_REC.CLASS
                    WHEN    '01'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS.1';
                    WHEN    '02'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS.2';
                    WHEN    '03'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS.3';
                    WHEN    '04'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS.4';
                    WHEN    '05'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS.5';
                    WHEN    '06'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS.6';
                    ELSE    V_KEY_VAL := 'NA';
                    END CASE;
                    V_BUNDLE_REPORT_V3 := PCRD_REPORTING_TOOLS_V3.NEW_BUNDLE_REPORT_V3;
                    V_BUNDLE_REPORT_V3.KEY_VAL                      := V_KEY_VAL;
                    V_BUNDLE_REPORT_V3.LANGUAGE_1                   := V_STG_TRANS_STATISTICS_DC_REC.LANGUAGE_1;
                    V_BUNDLE_REPORT_V3.LANGUAGE_2                   := V_STG_TRANS_STATISTICS_DC_REC.LANGUAGE_2;
                    V_BUNDLE_REPORT_V3.LANGUAGE_3                   := V_STG_TRANS_STATISTICS_DC_REC.LANGUAGE_3;
                    RETURN_STATUS   :=  PCRD_REPORTING_TOOLS_V3.GET_REPORT_LABELS   ( V_BUNDLE_REPORT_V3 );
                    IF RETURN_STATUS <> DECLARATION_CST.OK
                    THEN
                        V_ENV_INFO_TRACE.USER_MESSAGE   :=  'ERROR RETURNED BY PCRD_REPORTING_TOOLS_V3.GET_REPORT_LABELS'  ||
                                                            ',BUNDLE : ' || PCRD_REPORTING_TOOLS_V3.BUNDLE ||
                                                            ',KEY_VAL : ' || V_KEY_VAL;
                        PCRD_GENERAL_TOOLS.PUT_TRACES  (V_ENV_INFO_TRACE, $$PLSQL_LINE  );
                        RETURN  ( DECLARATION_CST.ERROR );
                    END IF;
                    V_STG_TRANS_STATISTICS_DC_REC.CLASS_DESCRIPTION_1            :=  V_BUNDLE_REPORT_V3.VALUE_1;
                    V_STG_TRANS_STATISTICS_DC_REC.CLASS_DESCRIPTION_2            :=  V_BUNDLE_REPORT_V3.VALUE_2;
                    V_STG_TRANS_STATISTICS_DC_REC.CLASS_DESCRIPTION_3            :=  V_BUNDLE_REPORT_V3.VALUE_3;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_SALES_TRX              := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_SALES_TRX            := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_SALES_ONUS_TRX         := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_SALES_ONUS_TRX       := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_SALES_NAT_IN_TRX       := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_SALES_NAT_IN_TRX     := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_SALES_INTL_TRX         := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_SALES_INTL_TRX       := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_SALES_EUR_TRX          := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_SALES_EUR_TRX        := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CASH_TRX               := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_CASH_TRX             := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CASH_ONUS_TRX          := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_CASH_ONUS_TRX        := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CASH_NAT_IN_TRX        := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_CASH_NAT_IN_TRX      := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CASH_INTL_TRX          := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_CASH_INTL_TRX        := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CASH_EUR_TRX           := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_CASH_EUR_TRX         := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_ATM_TRX                := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_ATM_TRX              := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_ATM_ONUS_TRX           := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_ATM_ONUS_TRX         := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_ATM_NAT_IN_TRX         := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_ATM_NAT_IN_TRX       := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_ATM_INTL_TRX           := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_ATM_INTL_TRX         := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_ATM_EUR_TRX            := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_ATM_EUR_TRX          := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_ALL_TRX                := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_ALL_TRX              := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_ALL_ONUS_TRX           := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_ALL_ONUS_TRX         := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_ALL_NAT_IN_TRX         := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_ALL_NAT_IN_TRX       := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_ALL_INTL_TRX           := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_ALL_INTL_TRX         := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_ALL_EUR_TRX            := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_ALL_EUR_TRX          := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_PUR_CHIP_TRX           := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_PUR_CHIP_TRX           := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_PUR_CHIP_ONUS_TRX      := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_PUR_CHIP_ONUS_TRX      := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_PUR_CHIP_NAT_IN_TRX    := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_PUR_CHIP_NAT_IN_TRX    := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_PUR_CHIP_INTL_TRX      := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_PUR_CHIP_INTL_TRX    := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_PUR_CHIP_EUR_TRX       := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_PUR_CHIP_EUR_TRX     := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_PUR_ECOM_TRX           := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_PUR_ECOM_TRX           := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_PUR_ECOM_ONUS_TRX      := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_PUR_ECOM_ONUS_TRX      := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_PUR_ECOM_NAT_IN_TRX    := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_PUR_ECOM_NAT_IN_TRX    := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_PUR_ECOM_INTL_TRX      := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_PUR_ECOM_INTL_TRX    := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_PUR_ECOM_EUR_TRX       := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_PUR_ECOM_EUR_TRX     := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CR_VOUCHER_TRX         := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_CR_VOUCHER_TRX       := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CR_VOUCHER_ONUS_TRX    := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_CR_VOUCHER_ONUS_TRX    := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CR_VOUCHER_NAT_TRX     := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_CR_VOUCHER_NAT_TRX     := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CR_VOUCHER_INTL_TRX    := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_CR_VOUCHER_INTL_TRX    := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_ORG_CR_TRX             := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_ORG_CR_TRX           := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_ORG_CR_ONUS_TRX        := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_ORG_CR_ONUS_TRX        := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_ORG_CR_NAT_TRX         := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_ORG_CR_NAT_TRX         := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_ORG_CR_INTL_TRX        := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_ORG_CR_INTL_TRX        := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_AFT_TRX                := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_AFT_TRX              := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_AFT_ONUS_TRX           := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_AFT_ONUS_TRX           := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_AFT_NAT_TRX            := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_AFT_NAT_TRX            := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_AFT_INTL_TRX           := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_AFT_INTL_TRX           := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CASHBACK_TRX           := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_CASHBACK_TRX           := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CASHBACK_ONUS_TRX      := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_CASHBACK_ONUS_TRX      := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CASHBACK_NAT_IN_TRX    := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_CASHBACK_NAT_IN_TRX    := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CASHBACK_INTL_TRX      := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_CASHBACK_INTL_TRX    := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CASHBACK_EUR_TRX       := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_CASHBACK_EUR_TRX     := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_BAL_TRNS_TRX           := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_BAL_TRNS_TRX           := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_BAL_TRNS_ONUS_TRX      := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_BAL_TRNS_ONUS_TRX      := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_BAL_TRNS_NAT_IN_TRX    := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_BAL_TRNS_NAT_IN_TRX    := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_BAL_TRNS_INTL_TRX      := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_BAL_TRNS_INTL_TRX    := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_BAL_TRNS_EUR_TRX       := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_BAL_TRNS_EUR_TRX     := 0.;
                    -- START HFZ20221114
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CRD_PRS_SLS_ONUS_TRX   := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_CRD_PRS_SLS_ONUS_TRX   := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CRD_PRS_SLS_NAT_IN_TRX := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_CRD_PRS_SLS_NAT_IN_TRX := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CRD_PRS_SLS_INTL_TRX   := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_CRD_PRS_SLS_INTL_TRX   := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CD_NT_PRS_SLS_ONUS_TRX := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_CD_NT_PRS_SLS_ONUS_TRX := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NB_CD_NT_PRS_SLS_NAT_IN_TX := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VL_CD_NT_PRS_SLS_NAT_IN_TX := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CD_NT_PRS_SLS_INTL_TRX := 0.;
                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_CD_NT_PRS_SLS_INTL_TRX := 0.;
                    -- END   HFZ20221114
                    FOR V_M_STAT_TRANSACTIONS_REC IN CUR_M_STAT_TRANSACTIONS (P_BANK_RECORD.BANK_CODE,P_NETWORK_RECORD.NETWORK_CODE,P_CLASS,P_CLASS_GROUP,V_START_DATE)
                    LOOP
                        --START MBH10012018
                        IF(V_M_STAT_TRANSACTIONS_REC.CARDHOLDER_TRANSACTION_SIGN = 'D')
                        THEN
                            V_TRANSACTION_AMOUNT_UC :=  V_M_STAT_TRANSACTIONS_REC.TRANSACTION_AMOUNT_UC;--V_M_STAT_TRANSACTIONS_REC.TRANSACTION_AMOUNT_LC;
                        ELSE
                            V_TRANSACTION_AMOUNT_UC :=  V_M_STAT_TRANSACTIONS_REC.TRANSACTION_AMOUNT_UC * -1;--V_M_STAT_TRANSACTIONS_REC.TRANSACTION_AMOUNT_LC * -1
                        END IF;
                        V_STG_TRANS_STATISTICS_DC_REC.CURRENCY_CODE        :=  V_M_STAT_TRANSACTIONS_REC.USER_CURRENCY_CODE;--V_M_STAT_TRANSACTIONS_REC.LOCAL_CURRENCY_CODE;
                        V_STG_TRANS_STATISTICS_DC_REC.CURRENCY_EXPONENT    :=  V_M_STAT_TRANSACTIONS_REC.USER_CURRENCY_EXP;--V_M_STAT_TRANSACTIONS_REC.LOCAL_CURRENCY_EXP;
                        --END MBH10012018
                        IF (V_M_STAT_TRANSACTIONS_REC.SETTLEMENT_FLAG IS NOT NULL)
                        THEN
                            IF V_M_STAT_TRANSACTIONS_REC.SETTLEMENT_FLAG  = '8'  OR V_M_STAT_TRANSACTIONS_REC.ORIGIN_CODE = GLOBAL_VARS.ORIGIN_CODE_C_ONUS_M_NATIONAL -- TED02062016
                            THEN
                                IF  (   V_M_STAT_TRANSACTIONS_REC.TRANSACTION_CODE  IN (GLOBAL_VARS.TC_PURCHASE,GLOBAL_VARS.TC_UNIQUE))
                                THEN
                                -- START HFZ20221114
                                    IF (SUBSTR(V_M_STAT_TRANSACTIONS_REC.POS_DATA, 6, 1) = '1')
                                    THEN
                                         V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_CRD_PRS_SLS_NAT_IN_TRX   := NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_CRD_PRS_SLS_NAT_IN_TRX,0)
                                                                                                            +   V_TRANSACTION_AMOUNT_UC;
                                         V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CRD_PRS_SLS_NAT_IN_TRX   := NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CRD_PRS_SLS_NAT_IN_TRX,0)
                                                                                                            +   V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION;
                                    ELSE
                                        V_STG_TRANS_STATISTICS_DC_REC.TOT_NB_CD_NT_PRS_SLS_NAT_IN_TX    := NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NB_CD_NT_PRS_SLS_NAT_IN_TX,0)
                                                                                                            +   V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION;
                                        V_STG_TRANS_STATISTICS_DC_REC.TOT_VL_CD_NT_PRS_SLS_NAT_IN_TX    := NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VL_CD_NT_PRS_SLS_NAT_IN_TX,0)
                                                                                                            +   V_TRANSACTION_AMOUNT_UC;
                                    END IF ;--END HFZ20221114
                                    IF (NVL(V_M_STAT_TRANSACTIONS_REC.POS_ENTRY_MODE,'X') = 'C')
                                    THEN
                                            V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_PUR_CHIP_NAT_IN_TRX     :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_PUR_CHIP_NAT_IN_TRX,0)
                                                                                                        +   V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION;
                                            V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_PUR_CHIP_NAT_IN_TRX     :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_PUR_CHIP_NAT_IN_TRX,0)
                                                                                                        +   V_TRANSACTION_AMOUNT_UC;
                                    ELSIF (NVL(V_M_STAT_TRANSACTIONS_REC.POS_ENTRY_MODE,'X') = 'E')
                                    THEN
                                            V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_PUR_ECOM_NAT_IN_TRX     :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_PUR_ECOM_NAT_IN_TRX,0)
                                                                                                        +   V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION;
                                            V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_PUR_ECOM_NAT_IN_TRX     :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_PUR_ECOM_NAT_IN_TRX,0)
                                                                                                        +   V_TRANSACTION_AMOUNT_UC;
                                    ELSE
                                            V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_SALES_NAT_IN_TRX         :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_SALES_NAT_IN_TRX,0)
                                                                                                                +   V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION;
                                            V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_SALES_NAT_IN_TRX       :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_SALES_NAT_IN_TRX,0)
                                                                                                                +   V_TRANSACTION_AMOUNT_UC;
                                    END IF;
                                END IF;
                                IF   (   V_M_STAT_TRANSACTIONS_REC.MCC   =   '6010' )
                                THEN
                                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CASH_NAT_IN_TRX          :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CASH_NAT_IN_TRX,0)
                                                                                                    +   V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION;
                                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_CASH_NAT_IN_TRX        :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_CASH_NAT_IN_TRX,0)
                                                                                                    +   V_TRANSACTION_AMOUNT_UC;
                                END IF;
                                IF   ( V_M_STAT_TRANSACTIONS_REC.TRANSACTION_CODE = GLOBAL_VARS.TC_WITHDRAWAL)
                                THEN
                                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_ATM_NAT_IN_TRX           :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_ATM_NAT_IN_TRX,0)
                                                                                                    +   V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION;
                                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_ATM_NAT_IN_TRX         :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_ATM_NAT_IN_TRX,0)
                                                                                                    +   V_TRANSACTION_AMOUNT_UC;

                                END IF;
                                IF (V_M_STAT_TRANSACTIONS_REC.TRANSACTION_CODE = GLOBAL_VARS.TC_CREDIT_VOUCHER)
                                THEN
                                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CR_VOUCHER_NAT_TRX       :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CR_VOUCHER_NAT_TRX,0)
                                                                                                    +   V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION;
                                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_CR_VOUCHER_NAT_TRX       :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_CR_VOUCHER_NAT_TRX,0)
                                                                                                    +   V_TRANSACTION_AMOUNT_UC;
                                END IF;
                                V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_ALL_NAT_IN_TRX               :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_SALES_NAT_IN_TRX,0)
                                                                                                    +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_PUR_CHIP_NAT_IN_TRX,0)
                                                                                                    +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_PUR_ECOM_NAT_IN_TRX,0)
                                                                                                    +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CASH_NAT_IN_TRX,0)
                                                                                                    +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_ATM_NAT_IN_TRX,0)
                                                                                                    +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CASHBACK_NAT_IN_TRX,0)
                                                                                                    +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_ORG_CR_NAT_TRX,0)
                                                                                                    +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CR_VOUCHER_NAT_TRX,0)
                                                                                                    +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_AFT_NAT_TRX,0)
                                                                                                    +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_BAL_TRNS_NAT_IN_TRX,0);

                                V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_ALL_NAT_IN_TRX             :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_SALES_NAT_IN_TRX,0)
                                                                                                    +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_PUR_CHIP_NAT_IN_TRX,0)
                                                                                                    +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_PUR_ECOM_NAT_IN_TRX,0)
                                                                                                    +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_CASH_NAT_IN_TRX,0)
                                                                                                    +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_ATM_NAT_IN_TRX,0)
                                                                                                    +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_CASHBACK_NAT_IN_TRX,0)
                                                                                                    +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_CR_VOUCHER_NAT_TRX,0)
                                                                                                    +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_AFT_NAT_TRX,0)
                                                                                                    +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_BAL_TRNS_NAT_IN_TRX,0)
                                                                                                    +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_ORG_CR_NAT_TRX,0);

                                ELSIF V_M_STAT_TRANSACTIONS_REC.SETTLEMENT_FLAG  = '0'
                                THEN
                                    IF  (   V_M_STAT_TRANSACTIONS_REC.TRANSACTION_CODE  IN (GLOBAL_VARS.TC_PURCHASE,GLOBAL_VARS.TC_UNIQUE) )
                                    THEN
                                    -- START HFZ20221114
                                        IF (SUBSTR(V_M_STAT_TRANSACTIONS_REC.POS_DATA, 6, 1) = '1')
                                        THEN
                                             V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_CRD_PRS_SLS_INTL_TRX    :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_CRD_PRS_SLS_INTL_TRX,0)
                                                                                                            +   V_TRANSACTION_AMOUNT_UC;
                                             V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CRD_PRS_SLS_INTL_TRX        := NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CRD_PRS_SLS_INTL_TRX,0)
                                                                                                            +   V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION;
                                        ELSE
                                            V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CD_NT_PRS_SLS_INTL_TRX    := NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CD_NT_PRS_SLS_INTL_TRX,0)
                                                                                                            +   V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION;
                                            V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_CD_NT_PRS_SLS_INTL_TRX    :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_CD_NT_PRS_SLS_INTL_TRX,0)
                                                                                                            +   V_TRANSACTION_AMOUNT_UC;
                                        END IF ;--END HFZ20221114
                                        IF (NVL(V_M_STAT_TRANSACTIONS_REC.POS_ENTRY_MODE,'X') = 'C')
                                        THEN
                                            V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_PUR_CHIP_INTL_TRX     :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_PUR_CHIP_INTL_TRX,0)
                                                                                                        +   V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION;
                                            V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_PUR_CHIP_INTL_TRX     :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_PUR_CHIP_INTL_TRX,0)
                                                                                                        +   V_TRANSACTION_AMOUNT_UC;
                                        ELSIF (NVL(V_M_STAT_TRANSACTIONS_REC.POS_ENTRY_MODE,'X') = 'E')
                                        THEN
                                                V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_PUR_ECOM_INTL_TRX       :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_PUR_ECOM_INTL_TRX,0)
                                                                                                            +   V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION;
                                                V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_PUR_ECOM_INTL_TRX     :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_PUR_ECOM_INTL_TRX,0)
                                                                                                            +   V_TRANSACTION_AMOUNT_UC;
                                        ELSE
                                                V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_SALES_INTL_TRX           :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_SALES_INTL_TRX,0)
                                                                                                            +   V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION;
                                                V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_SALES_INTL_TRX         :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_SALES_INTL_TRX,0)
                                                                                                            +   V_TRANSACTION_AMOUNT_UC;
                                        END IF;
                                    END IF;
                                    IF   (   V_M_STAT_TRANSACTIONS_REC.MCC   =   '6010')
                                    THEN
                                        V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CASH_INTL_TRX            :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CASH_INTL_TRX,0)
                                                                                                        +   V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION;
                                        V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_CASH_INTL_TRX          := NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_CASH_INTL_TRX,0)
                                                                                                        +   V_TRANSACTION_AMOUNT_UC;
                                    END IF;
                                    IF   (V_M_STAT_TRANSACTIONS_REC.TRANSACTION_CODE = GLOBAL_VARS.TC_WITHDRAWAL) --SAB06102015
                                    THEN
                                        V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_ATM_INTL_TRX             :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_ATM_INTL_TRX,0)
                                                                                                        +   V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION;
                                        V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_ATM_INTL_TRX           :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_ATM_INTL_TRX,0)
                                                                                                        +   V_TRANSACTION_AMOUNT_UC;
                                    END IF;
                                    /*IF   ( V_M_STAT_TRANSACTIONS_REC.TRANSACTION_CODE = GLOBAL_VARS.TC_PAYMENT)
                                    THEN
                                        V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_ORG_CR_INTL_TRX           :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_ORG_CR_INTL_TRX,0)
                                                                                                        +   V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION;
                                        V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_ORG_CR_INTL_TRX           :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_ORG_CR_INTL_TRX,0)
                                                                                                        +   V_TRANSACTION_AMOUNT_LC;

                                    END IF;*/
                                    IF (V_M_STAT_TRANSACTIONS_REC.TRANSACTION_CODE = GLOBAL_VARS.TC_CREDIT_VOUCHER)
                                    THEN
                                        V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CR_VOUCHER_INTL_TRX       :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CR_VOUCHER_INTL_TRX,0)
                                                                                                        +   V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION;
                                        V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_CR_VOUCHER_INTL_TRX       :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_CR_VOUCHER_INTL_TRX,0)
                                                                                                        +   V_TRANSACTION_AMOUNT_UC;
                                    END IF;
                                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_ALL_INTL_TRX                  :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_SALES_INTL_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_PUR_CHIP_INTL_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_PUR_ECOM_INTL_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CASH_INTL_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_ATM_INTL_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CASHBACK_INTL_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_ORG_CR_INTL_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CR_VOUCHER_INTL_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_AFT_INTL_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_BAL_TRNS_INTL_TRX,0);

                                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_ALL_INTL_TRX                :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_SALES_INTL_TRX,0)
                                                                                                         +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_PUR_CHIP_INTL_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_PUR_ECOM_INTL_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_CASH_INTL_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_ATM_INTL_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_CASHBACK_INTL_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_CR_VOUCHER_INTL_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_AFT_INTL_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_BAL_TRNS_INTL_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_ORG_CR_INTL_TRX,0);
                                END IF;
                            ELSE
                                IF V_M_STAT_TRANSACTIONS_REC.ORIGIN_CODE    =   GLOBAL_VARS.ORIGIN_CODE_C_ONUS_M_LOCAL
                                THEN
                                    IF  (   V_M_STAT_TRANSACTIONS_REC.TRANSACTION_CODE  IN (GLOBAL_VARS.TC_PURCHASE,GLOBAL_VARS.TC_UNIQUE))
                                    THEN
                                    -- START HFZ20221114
                                        IF (SUBSTR(V_M_STAT_TRANSACTIONS_REC.POS_DATA, 6, 1) = '1')
                                        THEN
                                             V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_CRD_PRS_SLS_ONUS_TRX     :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_CRD_PRS_SLS_ONUS_TRX,0)
                                                                                                            +   V_TRANSACTION_AMOUNT_UC;
                                             V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CRD_PRS_SLS_ONUS_TRX     :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CRD_PRS_SLS_ONUS_TRX,0)
                                                                                                            +   V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION;
                                        ELSE
                                            V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CD_NT_PRS_SLS_ONUS_TRX    :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CD_NT_PRS_SLS_ONUS_TRX,0)
                                                                                                            +   V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION;
                                            V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_CD_NT_PRS_SLS_ONUS_TRX    :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_CD_NT_PRS_SLS_ONUS_TRX,0)
                                                                                                            +   V_TRANSACTION_AMOUNT_UC;
                                        END IF ;--END HFZ20221114
                                        IF (NVL(V_M_STAT_TRANSACTIONS_REC.POS_ENTRY_MODE,'X') = 'C')
                                        THEN
                                            V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_PUR_CHIP_ONUS_TRX     :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_PUR_CHIP_ONUS_TRX,0)
                                                                                                        +   V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION;
                                            V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_PUR_CHIP_ONUS_TRX     :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_PUR_CHIP_ONUS_TRX,0)
                                                                                                        +   V_TRANSACTION_AMOUNT_UC;
                                        ELSIF (NVL(V_M_STAT_TRANSACTIONS_REC.POS_ENTRY_MODE,'X') = 'E')
                                        THEN
                                                V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_PUR_ECOM_ONUS_TRX       :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_PUR_ECOM_ONUS_TRX,0)
                                                                                                            +   V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION;
                                                V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_PUR_ECOM_ONUS_TRX       :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_PUR_ECOM_ONUS_TRX,0)
                                                                                                            +   V_TRANSACTION_AMOUNT_UC;
                                        ELSE
                                                V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_SALES_ONUS_TRX           :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_SALES_ONUS_TRX,0)
                                                                                                            +   NVL(V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION,0);
                                                V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_SALES_ONUS_TRX         :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_SALES_ONUS_TRX,0)
                                                                                                            +   NVL(V_TRANSACTION_AMOUNT_UC,0);
                                        END IF;
                                    END IF;
                                    IF   (   V_M_STAT_TRANSACTIONS_REC.MCC   =   '6010' )
                                    THEN
                                        V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CASH_ONUS_TRX            :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CASH_ONUS_TRX,0)
                                                                                                        +   NVL(V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION,0);
                                        V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_CASH_ONUS_TRX          :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_CASH_ONUS_TRX,0)
                                                                                                        +   NVL(V_TRANSACTION_AMOUNT_UC,0);
                                    END IF;
                                    IF  (V_M_STAT_TRANSACTIONS_REC.TRANSACTION_CODE = GLOBAL_VARS.TC_WITHDRAWAL) --SAB06102015
                                    THEN
                                        V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_ATM_ONUS_TRX             :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_ATM_ONUS_TRX,0)
                                                                                                        +   NVL(V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION,0);
                                        V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_ATM_ONUS_TRX           :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_ATM_ONUS_TRX,0)
                                                                                                        +   NVL(V_TRANSACTION_AMOUNT_UC,0);
                                    END IF;
                                    IF (V_M_STAT_TRANSACTIONS_REC.TRANSACTION_CODE = GLOBAL_VARS.TC_CREDIT_VOUCHER)
                                    THEN
                                        V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CR_VOUCHER_ONUS_TRX       :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CR_VOUCHER_ONUS_TRX,0)
                                                                                                        +   V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION;
                                        V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_CR_VOUCHER_ONUS_TRX       :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_CR_VOUCHER_ONUS_TRX,0)
                                                                                                        +   V_TRANSACTION_AMOUNT_UC;
                                    END IF;
                                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_ALL_ONUS_TRX               :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_SALES_ONUS_TRX,0)
                                                                                                         +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_PUR_CHIP_ONUS_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_PUR_ECOM_ONUS_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CASH_ONUS_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_ATM_ONUS_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CASHBACK_ONUS_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_ORG_CR_ONUS_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CR_VOUCHER_ONUS_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_AFT_ONUS_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_BAL_TRNS_ONUS_TRX,0);

                                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_ALL_ONUS_TRX             :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_SALES_ONUS_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_PUR_CHIP_ONUS_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_PUR_ECOM_ONUS_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_CASH_ONUS_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_ATM_ONUS_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_CASHBACK_ONUS_TRX,0)
                                                                                                       -- +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_CR_VOUCHER_ONUS_TRX,0) ADD IN SECTION CREDITS_VOUCHER
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_AFT_ONUS_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_BAL_TRNS_ONUS_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_ORG_CR_ONUS_TRX,0);
                                ELSIF (V_M_STAT_TRANSACTIONS_REC.ORIGIN_CODE    =   GLOBAL_VARS.ORIGIN_CODE_C_ONUS_M_NATIONAL)
                                THEN
                                    IF  (   V_M_STAT_TRANSACTIONS_REC.TRANSACTION_CODE  IN (GLOBAL_VARS.TC_PURCHASE,GLOBAL_VARS.TC_UNIQUE))
                                    THEN
                                        IF (NVL(V_M_STAT_TRANSACTIONS_REC.POS_ENTRY_MODE,'X') = 'C')
                                        THEN
                                                V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_PUR_CHIP_NAT_IN_TRX     :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_PUR_CHIP_NAT_IN_TRX,0)
                                                                                                            +   V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION;
                                                V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_PUR_CHIP_NAT_IN_TRX     :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_PUR_CHIP_NAT_IN_TRX,0)
                                                                                                            +   V_TRANSACTION_AMOUNT_UC;
                                        ELSIF (NVL(V_M_STAT_TRANSACTIONS_REC.POS_ENTRY_MODE,'X') = 'E')
                                        THEN
                                                V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_PUR_ECOM_NAT_IN_TRX     :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_PUR_ECOM_NAT_IN_TRX,0)
                                                                                                            +   V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION;
                                                V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_PUR_ECOM_NAT_IN_TRX     :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_PUR_ECOM_NAT_IN_TRX,0)
                                                                                                            +   V_TRANSACTION_AMOUNT_UC;
                                        ELSE
                                                V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_SALES_NAT_IN_TRX         :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_SALES_NAT_IN_TRX,0)
                                                                                                            +   V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION;
                                                V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_SALES_NAT_IN_TRX       :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_SALES_NAT_IN_TRX,0)
                                                                                                            +   V_TRANSACTION_AMOUNT_UC;
                                        END IF;
                                    END IF;
                                    IF   (   V_M_STAT_TRANSACTIONS_REC.MCC   =   '6010' )
                                    THEN
                                        V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CASH_NAT_IN_TRX          :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CASH_NAT_IN_TRX,0)
                                                                                                        +   V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION;
                                        V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_CASH_NAT_IN_TRX        :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_CASH_NAT_IN_TRX,0)
                                                                                                        +   V_TRANSACTION_AMOUNT_UC;
                                    END IF;
                                    IF (V_M_STAT_TRANSACTIONS_REC.TRANSACTION_CODE = GLOBAL_VARS.TC_WITHDRAWAL) --SAB06102015
                                    THEN
                                        V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_ATM_NAT_IN_TRX           :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_ATM_NAT_IN_TRX,0)
                                                                                                        +   V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION;
                                        V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_ATM_NAT_IN_TRX         :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_ATM_NAT_IN_TRX,0)
                                                                                                        +   V_TRANSACTION_AMOUNT_UC;

                                    END IF;

                                    IF (V_M_STAT_TRANSACTIONS_REC.TRANSACTION_CODE = GLOBAL_VARS.TC_CREDIT_VOUCHER)
                                    THEN
                                        V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CR_VOUCHER_NAT_TRX       :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CR_VOUCHER_NAT_TRX,0)
                                                                                                        +   V_M_STAT_TRANSACTIONS_REC.NUMBER_TRANSACTION;
                                        V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_CR_VOUCHER_NAT_TRX       :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_CR_VOUCHER_NAT_TRX,0)
                                                                                                        +   V_TRANSACTION_AMOUNT_UC;
                                    END IF;
                                    V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_ALL_NAT_IN_TRX               :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_SALES_NAT_IN_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_PUR_CHIP_NAT_IN_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_PUR_ECOM_NAT_IN_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CASH_NAT_IN_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_ATM_NAT_IN_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CASHBACK_NAT_IN_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_ORG_CR_NAT_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CR_VOUCHER_NAT_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_AFT_NAT_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_BAL_TRNS_NAT_IN_TRX,0);

                                    V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_ALL_NAT_IN_TRX             :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_SALES_NAT_IN_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_PUR_CHIP_NAT_IN_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_PUR_ECOM_NAT_IN_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_CASH_NAT_IN_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_ATM_NAT_IN_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_CASHBACK_NAT_IN_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_CR_VOUCHER_NAT_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_AFT_NAT_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_BAL_TRNS_NAT_IN_TRX,0)
                                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_ORG_CR_NAT_TRX,0);                                    END IF;
                            END IF;
                            V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_SALES_TRX        :=     NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_SALES_ONUS_TRX,0)
                                                                                            +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_SALES_INTL_TRX,0)
                                                                                            +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_SALES_NAT_IN_TRX,0)
                                                                                            +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_SALES_EUR_TRX,0);

                            V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_SALES_TRX      :=      NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_SALES_ONUS_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_SALES_INTL_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_SALES_NAT_IN_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_SALES_EUR_TRX,0);

                            V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_PUR_CHIP_TRX     :=     NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_PUR_CHIP_ONUS_TRX,0)
                                                                                            +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_PUR_CHIP_NAT_IN_TRX,0)
                                                                                            +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_PUR_CHIP_INTL_TRX,0)
                                                                                            +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_PUR_CHIP_EUR_TRX,0);

                            V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_PUR_CHIP_TRX     :=      NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_PUR_CHIP_ONUS_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_PUR_CHIP_NAT_IN_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_PUR_CHIP_INTL_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_PUR_CHIP_EUR_TRX,0);

                            V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_PUR_ECOM_TRX     :=     NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_PUR_ECOM_ONUS_TRX,0)
                                                                                            +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_PUR_ECOM_NAT_IN_TRX,0)
                                                                                            +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_PUR_ECOM_INTL_TRX,0)
                                                                                            +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_PUR_ECOM_EUR_TRX,0);

                            V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_PUR_ECOM_TRX     :=      NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_PUR_ECOM_ONUS_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_PUR_ECOM_NAT_IN_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_PUR_ECOM_INTL_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_PUR_ECOM_EUR_TRX,0);

                            V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CASH_TRX        :=     NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CASH_ONUS_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CASH_INTL_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CASH_NAT_IN_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CASH_EUR_TRX,0);

                            V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_CASH_TRX      :=      NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_CASH_ONUS_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_CASH_INTL_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_CASH_NAT_IN_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_CASH_EUR_TRX,0);

                            V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_ATM_TRX          :=     NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_ATM_ONUS_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_ATM_INTL_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_ATM_NAT_IN_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_ATM_EUR_TRX,0);

                            V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_ATM_TRX        :=      NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_ATM_ONUS_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_ATM_INTL_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_ATM_NAT_IN_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_ATM_EUR_TRX,0);

                            V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CR_VOUCHER_TRX    :=     NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CR_VOUCHER_ONUS_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CR_VOUCHER_NAT_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CR_VOUCHER_INTL_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CR_VOUCHER_EUR_TRX,0);

                            V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_CR_VOUCHER_TRX  :=      NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_CR_VOUCHER_ONUS_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_CR_VOUCHER_NAT_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_CR_VOUCHER_INTL_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_CR_VOUCHER_EUR_TRX,0);

                            V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CASHBACK_TRX          :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CASHBACK_ONUS_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CASHBACK_INTL_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CASHBACK_NAT_IN_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_CASHBACK_EUR_TRX,0);

                            V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_CASHBACK_TRX        :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_CASHBACK_ONUS_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_CASHBACK_INTL_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_CASHBACK_NAT_IN_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_CASHBACK_EUR_TRX,0);

                            V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_BAL_TRNS_TRX          :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_BAL_TRNS_ONUS_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_BAL_TRNS_INTL_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_BAL_TRNS_NAT_IN_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_BAL_TRNS_EUR_TRX,0);

                            V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_BAL_TRNS_TRX        :=    NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_BAL_TRNS_ONUS_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_BAL_TRNS_INTL_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_BAL_TRNS_NAT_IN_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_BAL_TRNS_EUR_TRX,0);

                            V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_ORG_CR_TRX          :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_ORG_CR_ONUS_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_ORG_CR_INTL_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_ORG_CR_NAT_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_ORG_CR_EUR_TRX,0);

                            V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_ORG_CR_TRX        :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_ORG_CR_ONUS_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_ORG_CR_INTL_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_ORG_CR_NAT_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_ORG_CR_EUR_TRX,0);

                            V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_AFT_TRX          :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_AFT_ONUS_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_AFT_INTL_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_AFT_NAT_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_AFT_EUR_TRX,0);

                            V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_AFT_TRX        :=  NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_AFT_ONUS_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_AFT_INTL_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_AFT_NAT_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VAL_AFT_EUR_TRX,0);

                            V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_ALL_TRX             :=    NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_ALL_ONUS_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_ALL_NAT_IN_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_ALL_INTL_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_NBR_ALL_EUR_TRX,0);

                            V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_ALL_TRX           :=    NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_ALL_ONUS_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_ALL_NAT_IN_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_ALL_INTL_TRX,0)
                                                                                        +   NVL(V_STG_TRANS_STATISTICS_DC_REC.TOT_VALUE_ALL_EUR_TRX,0);


                    END LOOP;
                    IF (V_STG_TRANS_STATISTICS_DC_REC.CURRENCY_CODE IS NULL)
                    THEN
                        RETURN_STATUS   :=  PCRD_GET_PARAM_GENERAL_ROWS.GET_CURRENCY_TABLE (P_BANK_RECORD.BANK_CURRENCY_CODE,
                                                                                            V_CURRENCY_TABLE_RECORD,
                                                                                            TRUE);
                        IF RETURN_STATUS <> DECLARATION_CST.OK
                        THEN
                                V_ENV_INFO_TRACE.USER_MESSAGE := ':ERROR RETURNED BY PCRD_GET_PARAM_GENERAL_ROWS.GET_CURRENCY_TABLE';
                                PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
                                RETURN(RETURN_STATUS);
                        END IF;
                        V_STG_TRANS_STATISTICS_DC_REC.CURRENCY_CODE     :=  V_CURRENCY_TABLE_RECORD.CURRENCY_CODE;
                        V_STG_TRANS_STATISTICS_DC_REC.CURRENCY_EXPONENT :=  V_CURRENCY_TABLE_RECORD.CURRENCY_EXPONENT;
                    END IF;

                    RETURN_STATUS   :=  PCRD_STA_REPORTING_TOOLS_4_V3.PUT_ON_STG_TRANS_STATISTICS_DC(V_STG_TRANS_STATISTICS_DC_REC);
                    IF (RETURN_STATUS <> DECLARATION_CST.OK)
                    THEN
                        V_ENV_INFO_TRACE.USER_MESSAGE := 'ERROR RETURNED BY PCRD_STA_REPORTING_TOOLS_4_V3.PUT_ON_STG_TRANS_STATISTICS_DC STATISTICS_PERIOD = ' ||  V_STG_TRANS_STATISTICS_DC_REC.STATISTICS_PERIOD ||
                                                    ', BANK_CODE = ' || V_STG_TRANS_STATISTICS_DC_REC.BANK_CODE ||
                                                    ', NETWORK_CODE = ' || V_STG_TRANS_STATISTICS_DC_REC.NETWORK_CODE ||
                                                    ', CLASS_GROUP = ' || V_STG_TRANS_STATISTICS_DC_REC.CLASS_GROUP ||
                                                     ', CLASS = ' || V_STG_TRANS_STATISTICS_DC_REC.CLASS ;
                        PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
                        RETURN (DECLARATION_CST.ERROR);
                    END IF;
                    EXIT WHEN V_COMPTEUR = 3;
                    V_COMPTEUR := V_COMPTEUR + 1;
                    V_START_DATE    :=  ADD_MONTHS(V_START_DATE,1);
                END LOOP;
    RETURN(DECLARATION_CST.OK);
EXCEPTION WHEN OTHERS
THEN
    V_ENV_INFO_TRACE.USER_MESSAGE   :=  NULL;
    PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
    RETURN(DECLARATION_CST.NOK);
END PREP_STG_TRANSAC_STATISTICS_DC;
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
FUNCTION   PREP_STG_CARD_STATISTICS_DC   (  P_BUSINESS_DATE                     IN          DATE,
                                            P_CUTOFF_SEQUENCE                   IN          CUTOFF_FOLLOW_UP.CUTOFF_SEQUENCE%TYPE,
                                            P_LANGUAGE_1                        IN          PWC_BANK_REPORT_PARAMETERS.LANGUAGE_1%TYPE,
                                            P_LANGUAGE_2                        IN          PWC_BANK_REPORT_PARAMETERS.LANGUAGE_2%TYPE,
                                            P_LANGUAGE_3                        IN          PWC_BANK_REPORT_PARAMETERS.LANGUAGE_3%TYPE,
                                            P_BANK_RECORD                       IN          BANK%ROWTYPE,
                                            P_NETWORK_RECORD                    IN          NETWORK%ROWTYPE,
                                            P_CLASS                             IN          CARD_TYPE.CLASS%TYPE,
                                            P_CLASS_GROUP                       IN          CARD_TYPE.CLASS_GROUP%TYPE,
                                            P_START_DATE                        IN          DATE,
                                            P_END_DATE                          IN          DATE)
                                            RETURN PLS_INTEGER IS

RETURN_STATUS                           PLS_INTEGER;
V_START_DATE                            DATE;
V_COMPTEUR                              PLS_INTEGER;
V_STG_TRANS_CARD_DC_REC                 STG_CARD_STATISTICS_DC%ROWTYPE;
V_KEY_VAL                               RESSOURCE_BUNDLE.KEY_VAL%TYPE;
V_MULTI_LANG_REPORT_V3                  MULTI_LANG_REPORT_V3;
V_BUNDLE_REPORT_V3                      BUNDLE_REPORT_V3;
V_FEES_AMOUNT                           CARD_FEES.FEES_AMOUNT_FIRST%TYPE;
--V_FEES_AMOUNT_LC                        CARD_FEES.FEES_AMOUNT_FIRST%TYPE;
V_FEES_AMOUNT_UC                        CARD_FEES.FEES_AMOUNT_FIRST%TYPE;--MBH10012018
V_RATE_ISO_STR_FORMAT                   TRANS_BASE_TRANSACTION.CONV_RATE_SETTLEMENT%TYPE;
V_RATE_DATE                             DATE;
V_NBR_RECORD                            PLS_INTEGER:=0;
V_CONVERSION_DATE                       CONVERSION_RATE.RATE_DATE%TYPE;
V_MARKUP_AMOUNT                         TRANSACTION_HIST.CONVERSION_FEES%TYPE;
V_CURRENCY_TABLE_RECORD                 CURRENCY_TABLE%ROWTYPE;

CURSOR CUR_M_STAT_CARD (P_BANK_CODE BANK.BANK_CODE%TYPE,P_NETWORK_CODE  NETWORK.NETWORK_CODE%TYPE, P_CLASS CARD_TYPE.CLASS%TYPE, P_CLASS_GROUP CARD_TYPE.CLASS_GROUP%TYPE, P_START_DATE    DATE) IS
    SELECT   *
    FROM     M_STAT_CARD
    WHERE    STATISTICS_PERIOD         = TRUNC(TO_DATE(P_START_DATE),'MONTH')
    AND      BANK_CODE                 = P_BANK_CODE
    AND      CLASS                     = P_CLASS
    AND      CLASS_GROUP               = P_CLASS_GROUP
    AND      NETWORK_CODE              = P_NETWORK_CODE
    AND      CARD_PRODUCT_TYPE         = GLOBAL_VARS.DEBIT_CARD_TYPE;

CURSOR CUR_CARD_FEES_DEBIT (P_CLASS CARD_TYPE.CLASS%TYPE, P_CLASS_GROUP CARD_TYPE.CLASS_GROUP%TYPE)
IS
SELECT A.*
FROM CARD_FEES A, CARD_PRODUCT B, CARD_TYPE C
WHERE CARD_FEES_CODE = CARD_FEES_IND_PRIM
AND A.BANK_CODE = B.BANK_CODE
AND A.BANK_CODE = C.BANK_CODE
AND B.BANK_CODE = P_BANK_RECORD.BANK_CODE
AND B.NETWORK_CARD_TYPE = C.NETWORK_CARD_TYPE
AND C.CLASS             = P_CLASS
AND C.CLASS_GROUP       = P_CLASS_GROUP
AND B.NETWORK_CODE = P_NETWORK_RECORD.NETWORK_CODE
AND B.PRODUCT_TYPE = GLOBAL_VARS.DEBIT_CARD_TYPE
AND A.STATUS <> 'U';
BEGIN
      V_ENV_INFO_TRACE.USER_NAME      :=  GLOBAL_VARS.USER_NAME;
      V_ENV_INFO_TRACE.MODULE_CODE    :=  GLOBAL_VARS.ML_STATISTICS;
      V_ENV_INFO_TRACE.LANG           :=  GLOBAL_VARS.LANG;
      V_ENV_INFO_TRACE.PACKAGE_NAME   :=  $$PLSQL_UNIT;
      V_ENV_INFO_TRACE.FUNCTION_NAME  :=  'PREP_STG_CARD_STATISTICS_DC';

    V_START_DATE :=  TRUNC(P_START_DATE);
    V_COMPTEUR := 1;
    LOOP
        V_STG_TRANS_CARD_DC_REC                                :=  NULL;
        V_STG_TRANS_CARD_DC_REC.BANK_CODE                      :=  P_BANK_RECORD.BANK_CODE;
        V_STG_TRANS_CARD_DC_REC.BANK_NAME                      :=  P_BANK_RECORD.BANK_NAME;
        V_STG_TRANS_CARD_DC_REC.NETWORK_CODE                   :=  P_NETWORK_RECORD.NETWORK_CODE;
        V_STG_TRANS_CARD_DC_REC.NETWORK_NAME                   :=  P_NETWORK_RECORD.NETWORK_LABEL;
        V_STG_TRANS_CARD_DC_REC.BUSINESS_DATE                  :=  P_BUSINESS_DATE;
        V_STG_TRANS_CARD_DC_REC.CUTOFF_ID                      :=  P_CUTOFF_SEQUENCE;
        V_STG_TRANS_CARD_DC_REC.LANGUAGE_1                     :=  P_LANGUAGE_1;
        V_STG_TRANS_CARD_DC_REC.LANGUAGE_2                     :=  P_LANGUAGE_2;
        V_STG_TRANS_CARD_DC_REC.LANGUAGE_3                     :=  P_LANGUAGE_3;
        V_STG_TRANS_CARD_DC_REC.STATISTICS_PERIOD              :=  V_START_DATE;
        V_STG_TRANS_CARD_DC_REC.CLASS_GROUP                    :=  P_CLASS_GROUP;
        V_KEY_VAL := '';
        CASE    V_STG_TRANS_CARD_DC_REC.CLASS_GROUP
        WHEN    'IN'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS_GROUP.IN';
        WHEN    'BU'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS_GROUP.BU';
        ELSE    V_KEY_VAL := 'NA';
        END CASE;
        V_BUNDLE_REPORT_V3 := PCRD_REPORTING_TOOLS_V3.NEW_BUNDLE_REPORT_V3;
        V_BUNDLE_REPORT_V3.KEY_VAL                      := V_KEY_VAL;
        V_BUNDLE_REPORT_V3.LANGUAGE_1                   := V_STG_TRANS_CARD_DC_REC.LANGUAGE_1;
        V_BUNDLE_REPORT_V3.LANGUAGE_2                   := V_STG_TRANS_CARD_DC_REC.LANGUAGE_2;
        V_BUNDLE_REPORT_V3.LANGUAGE_3                   := V_STG_TRANS_CARD_DC_REC.LANGUAGE_3;
        RETURN_STATUS   :=  PCRD_REPORTING_TOOLS_V3.GET_REPORT_LABELS   ( V_BUNDLE_REPORT_V3 );
        IF RETURN_STATUS <> DECLARATION_CST.OK
        THEN
            V_ENV_INFO_TRACE.USER_MESSAGE   :=  'ERROR RETURNED BY PCRD_REPORTING_TOOLS_V3.GET_REPORT_LABELS'  ||
                                                ',BUNDLE : ' || PCRD_REPORTING_TOOLS_V3.BUNDLE ||
                                                ',KEY_VAL : ' || V_KEY_VAL;
            PCRD_GENERAL_TOOLS.PUT_TRACES  (V_ENV_INFO_TRACE, $$PLSQL_LINE  );
            RETURN  ( DECLARATION_CST.ERROR );
        END IF;
        V_STG_TRANS_CARD_DC_REC.CLASS_GROUP_DESC_1             :=  V_BUNDLE_REPORT_V3.VALUE_1;
        V_STG_TRANS_CARD_DC_REC.CLASS_GROUP_DESC_2             :=  V_BUNDLE_REPORT_V3.VALUE_2;
        V_STG_TRANS_CARD_DC_REC.CLASS_GROUP_DESC_3             :=  V_BUNDLE_REPORT_V3.VALUE_3;
        V_STG_TRANS_CARD_DC_REC.CLASS                          :=  P_CLASS;
        V_KEY_VAL := '';
        CASE    V_STG_TRANS_CARD_DC_REC.CLASS
        WHEN    '01'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS.1';
        WHEN    '02'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS.2';
        WHEN    '03'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS.3';
        WHEN    '04'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS.4';
        WHEN    '05'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS.5';
        WHEN    '06'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS.6';
        ELSE    V_KEY_VAL := 'NA';
        END CASE;
        V_BUNDLE_REPORT_V3 := PCRD_REPORTING_TOOLS_V3.NEW_BUNDLE_REPORT_V3;
        V_BUNDLE_REPORT_V3.KEY_VAL                      := V_KEY_VAL;
        V_BUNDLE_REPORT_V3.LANGUAGE_1                   := V_STG_TRANS_CARD_DC_REC.LANGUAGE_1;
        V_BUNDLE_REPORT_V3.LANGUAGE_2                   := V_STG_TRANS_CARD_DC_REC.LANGUAGE_2;
        V_BUNDLE_REPORT_V3.LANGUAGE_3                   := V_STG_TRANS_CARD_DC_REC.LANGUAGE_3;
        RETURN_STATUS   :=  PCRD_REPORTING_TOOLS_V3.GET_REPORT_LABELS   ( V_BUNDLE_REPORT_V3 );
        IF RETURN_STATUS <> DECLARATION_CST.OK
        THEN
            V_ENV_INFO_TRACE.USER_MESSAGE   :=  'ERROR RETURNED BY PCRD_REPORTING_TOOLS_V3.GET_REPORT_LABELS'  ||
                                                ',BUNDLE : ' || PCRD_REPORTING_TOOLS_V3.BUNDLE ||
                                                ',KEY_VAL : ' || V_KEY_VAL;
            PCRD_GENERAL_TOOLS.PUT_TRACES  (V_ENV_INFO_TRACE, $$PLSQL_LINE  );
            RETURN  ( DECLARATION_CST.ERROR );
        END IF;
        V_STG_TRANS_CARD_DC_REC.CLASS_DESCRIPTION_1            :=  V_BUNDLE_REPORT_V3.VALUE_1;
        V_STG_TRANS_CARD_DC_REC.CLASS_DESCRIPTION_2            :=  V_BUNDLE_REPORT_V3.VALUE_2;
        V_STG_TRANS_CARD_DC_REC.CLASS_DESCRIPTION_3            :=  V_BUNDLE_REPORT_V3.VALUE_3;
        V_STG_TRANS_CARD_DC_REC.NB_OF_CARDS                    :=   0.;
        V_STG_TRANS_CARD_DC_REC.NB_PRIMARY_CARDS               :=   0.;
        V_STG_TRANS_CARD_DC_REC.NB_SECONDARY_CARDS             :=   0.;
        V_STG_TRANS_CARD_DC_REC.NB_NEW_CARDS                   :=   0.;
        V_STG_TRANS_CARD_DC_REC.NB_RENEWAL_CARDS               :=   0.;
        V_STG_TRANS_CARD_DC_REC.NB_CANCELLED_CARDS             :=   0.;
        V_STG_TRANS_CARD_DC_REC.NB_INACTIVE_CARDS              :=   0.;
        V_STG_TRANS_CARD_DC_REC.NB_STOPPED_CARDS               :=   0.;
        V_STG_TRANS_CARD_DC_REC.NB_OF_DOUBLE_CARDS             :=   0.;
        V_STG_TRANS_CARD_DC_REC.NB_OF_CARD_REWARD              :=   0.;
        V_STG_TRANS_CARD_DC_REC.NB_OF_PINNED_CARD              :=   0.;
        V_STG_TRANS_CARD_DC_REC.NB_OF_CARD_CHIP                :=   0.;
        V_STG_TRANS_CARD_DC_REC.NB_OF_CRD_WITH_POS_ACTIVITY    :=   0.;
        V_STG_TRANS_CARD_DC_REC.NB_OF_CRD_WITH_ATM_ACTIVITY    :=   0.;
        V_STG_TRANS_CARD_DC_REC.NB_OF_CRD_WITH_ECOM_ACTIV      :=   0.;
        V_STG_TRANS_CARD_DC_REC.NB_OF_ACCOUNTS                 :=   0.;
        V_STG_TRANS_CARD_DC_REC.NB_OF_ACTIVE_ACCOUNTS          :=   0.;
        V_STG_TRANS_CARD_DC_REC.NB_OF_RESTRICTED_ACCOUNT       :=   0.;
        V_STG_TRANS_CARD_DC_REC.NB_OF_INTERNATIONAL_ACCOUNT    :=   0.;
        V_STG_TRANS_CARD_DC_REC.ANUAL_FEES_LC                  :=   0.;
        V_STG_TRANS_CARD_DC_REC.ANUAL_FEES_UC                  :=   0.;

        FOR V_M_STAT_CARD_REC IN CUR_M_STAT_CARD (P_BANK_RECORD.BANK_CODE,P_NETWORK_RECORD.NETWORK_CODE,P_CLASS,P_CLASS_GROUP,V_START_DATE)
        LOOP
           V_STG_TRANS_CARD_DC_REC.LOCAL_CURRENCY_CODE         :=  V_M_STAT_CARD_REC.LOCAL_CURRENCY_CODE;
           V_STG_TRANS_CARD_DC_REC.LOCAL_CURRENCY_EXPONENT     :=  V_M_STAT_CARD_REC.LOCAL_CURRENCY_EXPONENT;
           V_STG_TRANS_CARD_DC_REC.USER_CURRENCY_CODE          :=  V_M_STAT_CARD_REC.USER_CURRENCY_CODE;
           V_STG_TRANS_CARD_DC_REC.USER_CURRENCY_EXPONENT      :=  V_M_STAT_CARD_REC.USER_CURRENCY_EXPONENT;
           V_STG_TRANS_CARD_DC_REC.NB_OF_CARDS                 :=  V_STG_TRANS_CARD_DC_REC.NB_OF_CARDS + V_M_STAT_CARD_REC.NB_PRIMARY_CARDS + V_M_STAT_CARD_REC.NB_SECONDARY_CARDS;
           V_STG_TRANS_CARD_DC_REC.NB_PRIMARY_CARDS            :=  V_STG_TRANS_CARD_DC_REC.NB_PRIMARY_CARDS + V_M_STAT_CARD_REC.NB_PRIMARY_CARDS;
           V_STG_TRANS_CARD_DC_REC.NB_SECONDARY_CARDS          :=  V_STG_TRANS_CARD_DC_REC.NB_SECONDARY_CARDS + V_M_STAT_CARD_REC.NB_SECONDARY_CARDS;
           V_STG_TRANS_CARD_DC_REC.NB_NEW_CARDS                :=  V_STG_TRANS_CARD_DC_REC.NB_NEW_CARDS  + V_M_STAT_CARD_REC.NB_NEW_CARDS   ;
           V_STG_TRANS_CARD_DC_REC.NB_RENEWAL_CARDS            :=  V_STG_TRANS_CARD_DC_REC.NB_RENEWAL_CARDS + V_M_STAT_CARD_REC.NB_RENEWAL_CARDS;
           V_STG_TRANS_CARD_DC_REC.NB_CANCELLED_CARDS          :=  V_STG_TRANS_CARD_DC_REC.NB_CANCELLED_CARDS + V_M_STAT_CARD_REC.NB_CANCELLED_CARDS;
           V_STG_TRANS_CARD_DC_REC.NB_INACTIVE_CARDS           :=  V_STG_TRANS_CARD_DC_REC.NB_INACTIVE_CARDS + V_M_STAT_CARD_REC.NB_INACTIVE_CARDS;
           V_STG_TRANS_CARD_DC_REC.NB_STOPPED_CARDS            :=  V_STG_TRANS_CARD_DC_REC.NB_STOPPED_CARDS + V_M_STAT_CARD_REC.NB_STOPPED_CARDS;
           V_STG_TRANS_CARD_DC_REC.NB_OF_DOUBLE_CARDS          :=  V_STG_TRANS_CARD_DC_REC.NB_OF_DOUBLE_CARDS + V_M_STAT_CARD_REC.NB_OF_DOUBLE_CARDS;
           V_STG_TRANS_CARD_DC_REC.NB_OF_CARD_REWARD           :=  V_STG_TRANS_CARD_DC_REC.NB_OF_CARD_REWARD + V_M_STAT_CARD_REC.NB_OF_CARD_REWARD;
           V_STG_TRANS_CARD_DC_REC.NB_OF_PINNED_CARD           :=  V_STG_TRANS_CARD_DC_REC.NB_OF_PINNED_CARD + V_M_STAT_CARD_REC.NB_OF_PINNED_CARD;
           V_STG_TRANS_CARD_DC_REC.NB_OF_CARD_CHIP             :=  V_STG_TRANS_CARD_DC_REC.NB_OF_CARD_CHIP + V_M_STAT_CARD_REC.NB_OF_CARD_CHIP;
           V_STG_TRANS_CARD_DC_REC.NB_OF_CRD_WITH_POS_ACTIVITY :=  V_STG_TRANS_CARD_DC_REC.NB_OF_CRD_WITH_POS_ACTIVITY + V_M_STAT_CARD_REC.NB_OF_CRD_WITH_POS_ACTIVITY;
           V_STG_TRANS_CARD_DC_REC.NB_OF_CRD_WITH_ATM_ACTIVITY :=  V_STG_TRANS_CARD_DC_REC.NB_OF_CRD_WITH_ATM_ACTIVITY + V_M_STAT_CARD_REC.NB_OF_CRD_WITH_ATM_ACTIVITY;
           V_STG_TRANS_CARD_DC_REC.NB_OF_CRD_WITH_ECOM_ACTIV   :=  V_STG_TRANS_CARD_DC_REC.NB_OF_CRD_WITH_ECOM_ACTIV + V_M_STAT_CARD_REC.NB_OF_CRD_WITH_ECOM_ACTIV;
           V_STG_TRANS_CARD_DC_REC.NB_OF_ACCOUNTS              :=  V_STG_TRANS_CARD_DC_REC.NB_OF_ACCOUNTS + V_M_STAT_CARD_REC.NB_OF_ACCOUNTS;
           V_STG_TRANS_CARD_DC_REC.NB_OF_ACTIVE_ACCOUNTS       :=  V_STG_TRANS_CARD_DC_REC.NB_OF_ACTIVE_ACCOUNTS + V_M_STAT_CARD_REC.NB_OF_ACTIVE_ACCOUNTS;
        END LOOP;
        IF (V_STG_TRANS_CARD_DC_REC.LOCAL_CURRENCY_CODE IS NULL)
        THEN
            RETURN_STATUS   :=  PCRD_GET_PARAM_GENERAL_ROWS.GET_CURRENCY_TABLE (P_BANK_RECORD.BANK_CURRENCY_CODE,
                                                                                V_CURRENCY_TABLE_RECORD,
                                                                                TRUE);
            IF RETURN_STATUS <> DECLARATION_CST.OK
            THEN
                    V_ENV_INFO_TRACE.USER_MESSAGE := ':ERROR RETURNED BY PCRD_GET_PARAM_GENERAL_ROWS.GET_CURRENCY_TABLE';
                    PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
                    RETURN(RETURN_STATUS);
            END IF;
            V_STG_TRANS_CARD_DC_REC.LOCAL_CURRENCY_CODE     :=  V_CURRENCY_TABLE_RECORD.CURRENCY_CODE;
            V_STG_TRANS_CARD_DC_REC.LOCAL_CURRENCY_EXPONENT :=  V_CURRENCY_TABLE_RECORD.CURRENCY_EXPONENT;
        END IF;
        IF (V_COMPTEUR = 3)
        THEN
            FOR V_CARD_FEES_CREDIT_REC IN CUR_CARD_FEES_DEBIT (P_CLASS,P_CLASS_GROUP)
            LOOP
                IF (V_CARD_FEES_CREDIT_REC.FEES_AMOUNT_FIRST > 0)
                THEN
                    V_NBR_RECORD := V_NBR_RECORD + 1;
                    CASE V_CARD_FEES_CREDIT_REC.CARD_FEES_BILLING_PERIOD
                    WHEN 'Q' THEN V_FEES_AMOUNT := V_CARD_FEES_CREDIT_REC.FEES_AMOUNT_FIRST * 4;
                    WHEN 'M' THEN V_FEES_AMOUNT := V_CARD_FEES_CREDIT_REC.FEES_AMOUNT_FIRST * 12;
                    WHEN 'S' THEN V_FEES_AMOUNT := V_CARD_FEES_CREDIT_REC.FEES_AMOUNT_FIRST * 3;
                    WHEN 'Y' THEN V_FEES_AMOUNT := V_CARD_FEES_CREDIT_REC.FEES_AMOUNT_FIRST;
                    ELSE V_FEES_AMOUNT := 0;
                    END CASE;

                    RETURN_STATUS       :=  PCRD_CAI_GENERAL_TOOLS_1.CROSS_RATES_CONVERSION(GLOBAL_VARS.NETWORK_VISA,
                                                                                            V_CARD_FEES_CREDIT_REC.CURRENCY_CODE,
                                                                                            V_STG_TRANS_CARD_DC_REC.USER_CURRENCY_CODE,--MBH10012018 --P_BANK_RECORD.BANK_CURRENCY_CODE,
                                                                                            GLOBAL_VARS.RATE_ORIGIN_VISA,
                                                                                            GLOBAL_VARS.CONV_MODE_BAS_BUYING,
                                                                                            V_FEES_AMOUNT,
                                                                                            V_RATE_DATE,
                                                                                            V_RATE_ISO_STR_FORMAT,
                                                                                            V_FEES_AMOUNT_UC );

                    IF RETURN_STATUS = DECLARATION_CST.ERROR
                    THEN
                            V_ENV_INFO_TRACE.USER_MESSAGE := 'ERROR RETURNED BY PCRD_CAI_GENERAL_TOOLS_1.CROSS_RATES_CONVERSION:'         ||
                                                 'FROM CURRENCY :   '   ||  V_CARD_FEES_CREDIT_REC.CURRENCY_CODE    ||
                                                 'TO CURRENCY   :   '   ||  V_STG_TRANS_CARD_DC_REC.USER_CURRENCY_CODE;--MBH10012018 --P_BANK_RECORD.BANK_CURRENCY_CODE,
                            PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
                            RETURN DECLARATION_CST.ERROR;
                    ELSIF RETURN_STATUS = DECLARATION_CST.NOK
                    THEN
                        RETURN_STATUS       :=  PCRD_GENERAL_TOOLS.CONVERT_CURRENCY_AMOUNT( P_BANK_RECORD.BANK_CODE,
                                                                                            P_BANK_RECORD.BANK_CURRENCY_CODE,
                                                                                            V_CARD_FEES_CREDIT_REC.CURRENCY_CODE,
                                                                                            GLOBAL_VARS.RATE_ORIGIN_CENTER,
                                                                                            GLOBAL_VARS.CONV_MODE_BAS_BUYING,
                                                                                            V_STG_TRANS_CARD_DC_REC.USER_CURRENCY_CODE,--MBH10012018 --P_BANK_RECORD.BANK_CURRENCY_CODE,
                                                                                            V_FEES_AMOUNT,
                                                                                            P_BUSINESS_DATE,
                                                                                            NULL,
                                                                                            V_FEES_AMOUNT_UC,
                                                                                            V_CONVERSION_DATE,
                                                                                            V_RATE_ISO_STR_FORMAT,
                                                                                            V_MARKUP_AMOUNT,
                                                                                            FALSE,
                                                                                            NULL,
                                                                                            FALSE,
                                                                                            NULL );
                        IF RETURN_STATUS <> DECLARATION_CST.OK
                        THEN
                            V_ENV_INFO_TRACE.USER_MESSAGE    := 'ERROR CALLING PCRD_GENERAL_TOOLS.CONVERT_CURRENCY_AMOUNT '
                                                                ||' FROM CURRENCY : '           ||  V_CARD_FEES_CREDIT_REC.CURRENCY_CODE
                                                                ||'   TO  CURRENCY    :   '     ||  V_STG_TRANS_CARD_DC_REC.USER_CURRENCY_CODE;--MBH10012018 --P_BANK_RECORD.BANK_CURRENCY_CODE,
                            PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
                            RETURN(RETURN_STATUS);
                        END IF;
                    END IF;
                    V_STG_TRANS_CARD_DC_REC.ANUAL_FEES_LC := NVL(V_STG_TRANS_CARD_DC_REC.ANUAL_FEES_LC,0) + V_FEES_AMOUNT_UC;
                    V_STG_TRANS_CARD_DC_REC.ANUAL_FEES_UC := NVL(V_STG_TRANS_CARD_DC_REC.ANUAL_FEES_UC,0) + V_FEES_AMOUNT_UC; --MBH10012018
               END IF;
            END LOOP;
            IF (V_NBR_RECORD > 0)
            THEN
                V_STG_TRANS_CARD_DC_REC.ANUAL_FEES_LC := NVL(V_STG_TRANS_CARD_DC_REC.ANUAL_FEES_LC,0) / V_NBR_RECORD;
                V_STG_TRANS_CARD_DC_REC.ANUAL_FEES_UC := NVL(V_STG_TRANS_CARD_DC_REC.ANUAL_FEES_UC,0) / V_NBR_RECORD; --MBH10012018
            END IF;
        END IF;
        RETURN_STATUS   :=  PCRD_STA_REPORTING_TOOLS_1_V3.PUT_ON_STG_CARD_STATISTICS_DC(V_STG_TRANS_CARD_DC_REC);
        IF (RETURN_STATUS <> DECLARATION_CST.OK)
        THEN
            V_ENV_INFO_TRACE.USER_MESSAGE := 'ERROR RETURNED BY PCRD_STA_REPORTING_TOOLS_1_V3.PUT_ON_STG_CARD_STATISTICS_DC STATISTICS_PERIOD = ' ||  V_STG_TRANS_CARD_DC_REC.STATISTICS_PERIOD ||
                                ', BANK_CODE = ' || V_STG_TRANS_CARD_DC_REC.BANK_CODE ||
                                ', NETWORK_CODE = ' || V_STG_TRANS_CARD_DC_REC.NETWORK_CODE ||
                                ', CLASS_GROUP = ' || V_STG_TRANS_CARD_DC_REC.CLASS_GROUP ||
                                 ', CLASS = ' || V_STG_TRANS_CARD_DC_REC.CLASS ;
            PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
            RETURN (DECLARATION_CST.ERROR);
        END IF;
        EXIT WHEN V_COMPTEUR = 3;
        V_COMPTEUR := V_COMPTEUR + 1;
        V_START_DATE    :=  ADD_MONTHS(V_START_DATE,1);
    END LOOP;
    RETURN(DECLARATION_CST.OK);
EXCEPTION WHEN OTHERS
THEN
    V_ENV_INFO_TRACE.USER_MESSAGE   :=  NULL;
    PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
    RETURN(DECLARATION_CST.NOK);
END PREP_STG_CARD_STATISTICS_DC;
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
FUNCTION   PREP_STG_AUTH_STATISTICS_DC   (  P_BUSINESS_DATE                     IN          DATE,
                                            P_CUTOFF_SEQUENCE                   IN          CUTOFF_FOLLOW_UP.CUTOFF_SEQUENCE%TYPE,
                                            P_LANGUAGE_1                        IN          PWC_BANK_REPORT_PARAMETERS.LANGUAGE_1%TYPE,
                                            P_LANGUAGE_2                        IN          PWC_BANK_REPORT_PARAMETERS.LANGUAGE_2%TYPE,
                                            P_LANGUAGE_3                        IN          PWC_BANK_REPORT_PARAMETERS.LANGUAGE_3%TYPE,
                                            P_BANK_RECORD                       IN          BANK%ROWTYPE,
                                            P_NETWORK_RECORD                    IN          NETWORK%ROWTYPE,
                                            P_CLASS                             IN          CARD_TYPE.CLASS%TYPE,
                                            P_CLASS_GROUP                       IN          CARD_TYPE.CLASS_GROUP%TYPE,
                                            P_START_DATE                        IN          DATE)
                                            RETURN PLS_INTEGER IS

RETURN_STATUS                           PLS_INTEGER;
V_START_DATE                            DATE;
V_COMPTEUR                              PLS_INTEGER;
V_STG_AUTH_STATISTICS_DC_REC            STG_AUTH_STATISTICS_DC%ROWTYPE;
V_KEY_VAL                               RESSOURCE_BUNDLE.KEY_VAL%TYPE;
V_MULTI_LANG_REPORT_V3                  MULTI_LANG_REPORT_V3;
V_BUNDLE_REPORT_V3                      BUNDLE_REPORT_V3;
V_ACTION_LIST_RECORD                    ACTION_LIST%ROWTYPE;
V_CURRENCY_TABLE_RECORD                 CURRENCY_TABLE%ROWTYPE;
V_POS_ENTRY_MODE                        CHAR(1);

CURSOR CUR_M_STAT_AUTH_ACTIVITY (P_BANK_CODE BANK.BANK_CODE%TYPE,P_NETWORK_CODE  NETWORK.NETWORK_CODE%TYPE, P_CLASS CARD_TYPE.CLASS%TYPE, P_CLASS_GROUP CARD_TYPE.CLASS_GROUP%TYPE, P_START_DATE    DATE) IS
    SELECT   *
    FROM     M_STAT_AUTH_ACTIVITY
    WHERE    STATISTICS_PERIOD         = TRUNC(TO_DATE(P_START_DATE),'MONTH')
    AND      ISSUING_BANK              = P_BANK_CODE
    AND      NETWORK_CODE              = P_NETWORK_CODE
    AND      CLASS                     = P_CLASS
    AND      CLASS_GROUP               = P_CLASS_GROUP
    AND      CARD_PRODUCT_TYPE         = GLOBAL_VARS.DEBIT_CARD_TYPE;

BEGIN
      V_ENV_INFO_TRACE.USER_NAME      :=  GLOBAL_VARS.USER_NAME;
      V_ENV_INFO_TRACE.MODULE_CODE    :=  GLOBAL_VARS.ML_STATISTICS;
      V_ENV_INFO_TRACE.LANG           :=  GLOBAL_VARS.LANG;
      V_ENV_INFO_TRACE.PACKAGE_NAME   :=  $$PLSQL_UNIT;
      V_ENV_INFO_TRACE.FUNCTION_NAME  :=  'PREP_STG_AUTH_STATISTICS_DC';

    V_START_DATE :=  TRUNC(P_START_DATE);
    V_COMPTEUR := 1;
    LOOP
        V_STG_AUTH_STATISTICS_DC_REC                                :=  NULL;
        V_STG_AUTH_STATISTICS_DC_REC.BANK_CODE                      :=  P_BANK_RECORD.BANK_CODE;
        V_STG_AUTH_STATISTICS_DC_REC.BANK_NAME                      :=  P_BANK_RECORD.BANK_NAME;
        V_STG_AUTH_STATISTICS_DC_REC.NETWORK_CODE                   :=  P_NETWORK_RECORD.NETWORK_CODE;
        V_STG_AUTH_STATISTICS_DC_REC.NETWORK_NAME                   :=  P_NETWORK_RECORD.NETWORK_LABEL;
        V_STG_AUTH_STATISTICS_DC_REC.BUSINESS_DATE                  :=  P_BUSINESS_DATE;
        V_STG_AUTH_STATISTICS_DC_REC.CUTOFF_ID                      :=  P_CUTOFF_SEQUENCE;
        V_STG_AUTH_STATISTICS_DC_REC.LANGUAGE_1                     :=  P_LANGUAGE_1;
        V_STG_AUTH_STATISTICS_DC_REC.LANGUAGE_2                     :=  P_LANGUAGE_2;
        V_STG_AUTH_STATISTICS_DC_REC.LANGUAGE_3                     :=  P_LANGUAGE_3;
        V_STG_AUTH_STATISTICS_DC_REC.STATISTICS_PERIOD              :=  V_START_DATE;
        V_STG_AUTH_STATISTICS_DC_REC.CLASS_GROUP                    :=  P_CLASS_GROUP;
        V_KEY_VAL := '';
        CASE    V_STG_AUTH_STATISTICS_DC_REC.CLASS_GROUP
        WHEN    'IN'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS_GROUP.IN';
        WHEN    'BU'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS_GROUP.BU';
        ELSE    V_KEY_VAL := 'NA';
        END CASE;
        V_BUNDLE_REPORT_V3 := PCRD_REPORTING_TOOLS_V3.NEW_BUNDLE_REPORT_V3;
        V_BUNDLE_REPORT_V3.KEY_VAL                      := V_KEY_VAL;
        V_BUNDLE_REPORT_V3.LANGUAGE_1                   := V_STG_AUTH_STATISTICS_DC_REC.LANGUAGE_1;
        V_BUNDLE_REPORT_V3.LANGUAGE_2                   := V_STG_AUTH_STATISTICS_DC_REC.LANGUAGE_2;
        V_BUNDLE_REPORT_V3.LANGUAGE_3                   := V_STG_AUTH_STATISTICS_DC_REC.LANGUAGE_3;
        RETURN_STATUS   :=  PCRD_REPORTING_TOOLS_V3.GET_REPORT_LABELS   ( V_BUNDLE_REPORT_V3 );
        IF RETURN_STATUS <> DECLARATION_CST.OK
        THEN
            V_ENV_INFO_TRACE.USER_MESSAGE   :=  'ERROR RETURNED BY PCRD_REPORTING_TOOLS_V3.GET_REPORT_LABELS'  ||
                                                ',BUNDLE : ' || PCRD_REPORTING_TOOLS_V3.BUNDLE ||
                                                ',KEY_VAL : ' || V_KEY_VAL;
            PCRD_GENERAL_TOOLS.PUT_TRACES  (V_ENV_INFO_TRACE, $$PLSQL_LINE  );
            RETURN  ( DECLARATION_CST.ERROR );
        END IF;
        V_STG_AUTH_STATISTICS_DC_REC.CLASS_GROUP_DESC_1             :=  V_BUNDLE_REPORT_V3.VALUE_1;
        V_STG_AUTH_STATISTICS_DC_REC.CLASS_GROUP_DESC_2             :=  V_BUNDLE_REPORT_V3.VALUE_2;
        V_STG_AUTH_STATISTICS_DC_REC.CLASS_GROUP_DESC_3             :=  V_BUNDLE_REPORT_V3.VALUE_3;
        V_STG_AUTH_STATISTICS_DC_REC.CLASS                          :=  P_CLASS;
        V_KEY_VAL := '';
        CASE    V_STG_AUTH_STATISTICS_DC_REC.CLASS
        WHEN    '01'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS.1';
        WHEN    '02'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS.2';
        WHEN    '03'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS.3';
        WHEN    '04'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS.4';
        WHEN    '05'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS.5';
        WHEN    '06'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS.6';
        ELSE    V_KEY_VAL := 'NA';
        END CASE;
        V_BUNDLE_REPORT_V3 := PCRD_REPORTING_TOOLS_V3.NEW_BUNDLE_REPORT_V3;
        V_BUNDLE_REPORT_V3.KEY_VAL                      := V_KEY_VAL;
        V_BUNDLE_REPORT_V3.LANGUAGE_1                   := V_STG_AUTH_STATISTICS_DC_REC.LANGUAGE_1;
        V_BUNDLE_REPORT_V3.LANGUAGE_2                   := V_STG_AUTH_STATISTICS_DC_REC.LANGUAGE_2;
        V_BUNDLE_REPORT_V3.LANGUAGE_3                   := V_STG_AUTH_STATISTICS_DC_REC.LANGUAGE_3;
        RETURN_STATUS   :=  PCRD_REPORTING_TOOLS_V3.GET_REPORT_LABELS   ( V_BUNDLE_REPORT_V3 );
        IF RETURN_STATUS <> DECLARATION_CST.OK
        THEN
            V_ENV_INFO_TRACE.USER_MESSAGE   :=  'ERROR RETURNED BY PCRD_REPORTING_TOOLS_V3.GET_REPORT_LABELS'  ||
                                                ',BUNDLE : ' || PCRD_REPORTING_TOOLS_V3.BUNDLE ||
                                                ',KEY_VAL : ' || V_KEY_VAL;
            PCRD_GENERAL_TOOLS.PUT_TRACES  (V_ENV_INFO_TRACE, $$PLSQL_LINE  );
            RETURN  ( DECLARATION_CST.ERROR );
        END IF;
        V_STG_AUTH_STATISTICS_DC_REC.CLASS_DESCRIPTION_1            :=  V_BUNDLE_REPORT_V3.VALUE_1;
        V_STG_AUTH_STATISTICS_DC_REC.CLASS_DESCRIPTION_2            :=  V_BUNDLE_REPORT_V3.VALUE_2;
        V_STG_AUTH_STATISTICS_DC_REC.CLASS_DESCRIPTION_3            :=  V_BUNDLE_REPORT_V3.VALUE_3;
        V_STG_AUTH_STATISTICS_DC_REC.NB_AUTHORIZATION               :=   0;
        V_STG_AUTH_STATISTICS_DC_REC.TRANSACTION_AMOUNT_LC          :=   0.;
        V_STG_AUTH_STATISTICS_DC_REC.TRANSACTION_AMOUNT_UC          :=   0.;
        V_STG_AUTH_STATISTICS_DC_REC.CASH_BACK_AMOUNT_LC            :=   0.;
        V_STG_AUTH_STATISTICS_DC_REC.CASH_BACK_AMOUNT_UC            :=   0.;
        V_STG_AUTH_STATISTICS_DC_REC.BILLING_AMOUNT_LC              :=   0.;
        V_STG_AUTH_STATISTICS_DC_REC.BILLING_AMOUNT_UC              :=   0.;
        V_STG_AUTH_STATISTICS_DC_REC.ISS_SETTLEMENT_AMOUNT_LC       :=   0.;
        V_STG_AUTH_STATISTICS_DC_REC.ISS_SETTLEMENT_AMOUNT_UC       :=   0.;
        V_STG_AUTH_STATISTICS_DC_REC.ACQ_SETTLEMENT_AMOUNT_LC       :=   0.;
        V_STG_AUTH_STATISTICS_DC_REC.ACQ_SETTLEMENT_AMOUNT_UC       :=   0.;
        V_STG_AUTH_STATISTICS_DC_REC.TRANSACTION_FEE_LC             :=   0.;
        V_STG_AUTH_STATISTICS_DC_REC.TRANSACTION_FEE_UC             :=   0.;
        V_STG_AUTH_STATISTICS_DC_REC.ISS_SETTLEMENT_FEE_LC          :=   0.;
        V_STG_AUTH_STATISTICS_DC_REC.ISS_SETTLEMENT_FEE_UC          :=   0.;
        V_STG_AUTH_STATISTICS_DC_REC.ACQ_SETTLEMENT_FEE_LC          :=   0.;
        V_STG_AUTH_STATISTICS_DC_REC.ACQ_SETTLEMENT_FEE_UC          :=   0.;
        V_STG_AUTH_STATISTICS_DC_REC.NBR_PURCH_TRX_DEC_INS_FUND     := 0;
        V_STG_AUTH_STATISTICS_DC_REC.VAL_PURCH_TRX_DEC_INS_FUND     := 0;
        V_STG_AUTH_STATISTICS_DC_REC.NBR_PURCH_TRX_DEC_PICKUP       := 0;
        V_STG_AUTH_STATISTICS_DC_REC.VAL_PURCH_TRX_DEC_PICKUP       := 0;
        V_STG_AUTH_STATISTICS_DC_REC.NBR_PURCH_TRX_DEC_OTHERS       := 0;
        V_STG_AUTH_STATISTICS_DC_REC.VAL_PURCH_TRX_DEC_OTHERS       := 0;
        V_STG_AUTH_STATISTICS_DC_REC.NBR_P_ECOM_TRX_DEC_INS_FUND    := 0;
        V_STG_AUTH_STATISTICS_DC_REC.VAL_P_ECOM_TRX_DEC_INS_FUND    := 0;
        V_STG_AUTH_STATISTICS_DC_REC.NBR_P_ECOM_TRX_DEC_PICKUP      := 0;
        V_STG_AUTH_STATISTICS_DC_REC.VAL_P_ECOM_TRX_DEC_PICKUP      := 0;
        V_STG_AUTH_STATISTICS_DC_REC.NBR_P_ECOM_TRX_DEC_OTHERS      := 0;
        V_STG_AUTH_STATISTICS_DC_REC.VAL_P_ECOM_TRX_DEC_OTHERS      := 0;
        V_STG_AUTH_STATISTICS_DC_REC.NBR_P_CHIP_TRX_DEC_INS_FUND    := 0;
        V_STG_AUTH_STATISTICS_DC_REC.VAL_P_CHIP_TRX_DEC_INS_FUND    := 0;
        V_STG_AUTH_STATISTICS_DC_REC.NBR_P_CHIP_TRX_DEC_PICKUP      := 0;
        V_STG_AUTH_STATISTICS_DC_REC.VAL_P_CHIP_TRX_DEC_PICKUP      := 0;
        V_STG_AUTH_STATISTICS_DC_REC.NBR_P_CHIP_TRX_DEC_OTHERS      := 0;
        V_STG_AUTH_STATISTICS_DC_REC.VAL_P_CHIP_TRX_DEC_OTHERS      := 0;
        V_STG_AUTH_STATISTICS_DC_REC.NBR_ATM_TRX_DEC_INS_FUND       := 0;
        V_STG_AUTH_STATISTICS_DC_REC.VAL_ATM_TRX_DEC_INS_FUND       := 0;
        V_STG_AUTH_STATISTICS_DC_REC.NBR_ATM_TRX_DEC_PICKUP         := 0;
        V_STG_AUTH_STATISTICS_DC_REC.VAL_ATM_TRX_DEC_PICKUP         := 0;
        V_STG_AUTH_STATISTICS_DC_REC.NBR_ATM_TRX_DEC_OTHERS         := 0;
        V_STG_AUTH_STATISTICS_DC_REC.VAL_ATM_TRX_DEC_OTHERS         := 0;

        FOR V_M_STAT_AUTH_ACTIVITY_REC IN CUR_M_STAT_AUTH_ACTIVITY (P_BANK_RECORD.BANK_CODE,P_NETWORK_RECORD.NETWORK_CODE,P_CLASS,P_CLASS_GROUP,V_START_DATE)
        LOOP
            V_STG_AUTH_STATISTICS_DC_REC.LOCAL_CURRENCY_CODE            :=   V_M_STAT_AUTH_ACTIVITY_REC.LOCAL_CURRENCY_CODE;
            V_STG_AUTH_STATISTICS_DC_REC.LOCAL_CURRENCY_EXPONENT        :=   V_M_STAT_AUTH_ACTIVITY_REC.LOCAL_CURRENCY_EXP;
            V_STG_AUTH_STATISTICS_DC_REC.USER_CURRENCY_CODE             :=   V_M_STAT_AUTH_ACTIVITY_REC.USER_CURRENCY_CODE;
            V_STG_AUTH_STATISTICS_DC_REC.USER_CURRENCY_EXPONENT         :=   V_M_STAT_AUTH_ACTIVITY_REC.USER_CURRENCY_EXP;
            V_STG_AUTH_STATISTICS_DC_REC.NB_AUTHORIZATION               :=   V_STG_AUTH_STATISTICS_DC_REC.NB_AUTHORIZATION         + V_M_STAT_AUTH_ACTIVITY_REC.NB_AUTHORIZATION;
            V_STG_AUTH_STATISTICS_DC_REC.TRANSACTION_AMOUNT_LC          :=   V_STG_AUTH_STATISTICS_DC_REC.TRANSACTION_AMOUNT_LC    + V_M_STAT_AUTH_ACTIVITY_REC.TRANSACTION_AMOUNT_LC;
            V_STG_AUTH_STATISTICS_DC_REC.TRANSACTION_AMOUNT_UC          :=   V_STG_AUTH_STATISTICS_DC_REC.TRANSACTION_AMOUNT_UC    + V_M_STAT_AUTH_ACTIVITY_REC.TRANSACTION_AMOUNT_UC;
            V_STG_AUTH_STATISTICS_DC_REC.CASH_BACK_AMOUNT_LC            :=   V_STG_AUTH_STATISTICS_DC_REC.CASH_BACK_AMOUNT_LC      + V_M_STAT_AUTH_ACTIVITY_REC.CASH_BACK_AMOUNT_LC;
            V_STG_AUTH_STATISTICS_DC_REC.CASH_BACK_AMOUNT_UC            :=   V_STG_AUTH_STATISTICS_DC_REC.CASH_BACK_AMOUNT_UC      + V_M_STAT_AUTH_ACTIVITY_REC.CASH_BACK_AMOUNT_UC;
            V_STG_AUTH_STATISTICS_DC_REC.BILLING_AMOUNT_LC              :=   V_STG_AUTH_STATISTICS_DC_REC.BILLING_AMOUNT_LC        + V_M_STAT_AUTH_ACTIVITY_REC.BILLING_AMOUNT_LC;
            V_STG_AUTH_STATISTICS_DC_REC.BILLING_AMOUNT_UC              :=   V_STG_AUTH_STATISTICS_DC_REC.BILLING_AMOUNT_UC        + V_M_STAT_AUTH_ACTIVITY_REC.BILLING_AMOUNT_UC;
            V_STG_AUTH_STATISTICS_DC_REC.ISS_SETTLEMENT_AMOUNT_LC       :=   V_STG_AUTH_STATISTICS_DC_REC.ISS_SETTLEMENT_AMOUNT_LC + V_M_STAT_AUTH_ACTIVITY_REC.ISS_SETTLEMENT_AMOUNT_LC;
            V_STG_AUTH_STATISTICS_DC_REC.ISS_SETTLEMENT_AMOUNT_UC       :=   V_STG_AUTH_STATISTICS_DC_REC.ISS_SETTLEMENT_AMOUNT_UC + V_M_STAT_AUTH_ACTIVITY_REC.ISS_SETTLEMENT_AMOUNT_UC;
            V_STG_AUTH_STATISTICS_DC_REC.ACQ_SETTLEMENT_AMOUNT_LC       :=   V_STG_AUTH_STATISTICS_DC_REC.ACQ_SETTLEMENT_AMOUNT_LC + V_M_STAT_AUTH_ACTIVITY_REC.ACQ_SETTLEMENT_AMOUNT_LC;
            V_STG_AUTH_STATISTICS_DC_REC.ACQ_SETTLEMENT_AMOUNT_UC       :=   V_STG_AUTH_STATISTICS_DC_REC.ACQ_SETTLEMENT_AMOUNT_UC + V_M_STAT_AUTH_ACTIVITY_REC.ACQ_SETTLEMENT_AMOUNT_UC;

            RETURN_STATUS  := PCRD_GET_PARAM_GENERAL_ROWS.GET_ACTION_LIST ( V_M_STAT_AUTH_ACTIVITY_REC.ACTION_CODE,
                                                                            V_ACTION_LIST_RECORD);
            IF RETURN_STATUS  <> DECLARATION_CST.OK
            THEN
                V_ENV_INFO_TRACE.USER_MESSAGE   := 'ERROR RETURNED BY PCRD_GET_PARAM_GENERAL_ROWS.GET_ACTION_LIST ACTION_CODE :' || V_M_STAT_AUTH_ACTIVITY_REC.ACTION_CODE;
                PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
                --START MBH07042016
                --RETURN(DECLARATION_CST.NOK);
                V_ACTION_LIST_RECORD.CODE_ACTION := '909';
                V_ACTION_LIST_RECORD.ACTION_FLAG := 'D';
                --END MBH07042016
            END IF;
            IF (V_M_STAT_AUTH_ACTIVITY_REC.ACTION_CODE = '116'  AND SUBSTR(V_M_STAT_AUTH_ACTIVITY_REC.PROCESSING_CODE,1,2) = '01')
            THEN
                V_STG_AUTH_STATISTICS_DC_REC.NBR_ATM_TRX_DEC_INS_FUND  := V_STG_AUTH_STATISTICS_DC_REC.NBR_ATM_TRX_DEC_INS_FUND + V_M_STAT_AUTH_ACTIVITY_REC.NB_AUTHORIZATION;
                V_STG_AUTH_STATISTICS_DC_REC.VAL_ATM_TRX_DEC_INS_FUND  := V_STG_AUTH_STATISTICS_DC_REC.VAL_ATM_TRX_DEC_INS_FUND + V_M_STAT_AUTH_ACTIVITY_REC.TRANSACTION_AMOUNT_UC; --MBH10012018 --V_M_STAT_AUTH_ACTIVITY_REC.TRANSACTION_AMOUNT_LC;
            END IF;
            IF (V_ACTION_LIST_RECORD.ACTION_FLAG = 'P'  AND SUBSTR(V_M_STAT_AUTH_ACTIVITY_REC.PROCESSING_CODE,1,2) = '01')
            THEN
                V_STG_AUTH_STATISTICS_DC_REC.NBR_ATM_TRX_DEC_PICKUP  := V_STG_AUTH_STATISTICS_DC_REC.NBR_ATM_TRX_DEC_PICKUP + V_M_STAT_AUTH_ACTIVITY_REC.NB_AUTHORIZATION;
                V_STG_AUTH_STATISTICS_DC_REC.VAL_ATM_TRX_DEC_PICKUP  := V_STG_AUTH_STATISTICS_DC_REC.VAL_ATM_TRX_DEC_PICKUP + V_M_STAT_AUTH_ACTIVITY_REC.TRANSACTION_AMOUNT_UC; --MBH10012018 --V_M_STAT_AUTH_ACTIVITY_REC.TRANSACTION_AMOUNT_LC;
            END IF;
            IF (V_ACTION_LIST_RECORD.ACTION_FLAG <> 'A'  AND SUBSTR(V_M_STAT_AUTH_ACTIVITY_REC.PROCESSING_CODE,1,2) = '01' AND V_M_STAT_AUTH_ACTIVITY_REC.ACTION_CODE <> '116')
            THEN
                V_STG_AUTH_STATISTICS_DC_REC.NBR_ATM_TRX_DEC_OTHERS  := V_STG_AUTH_STATISTICS_DC_REC.NBR_ATM_TRX_DEC_OTHERS + V_M_STAT_AUTH_ACTIVITY_REC.NB_AUTHORIZATION;
                V_STG_AUTH_STATISTICS_DC_REC.VAL_ATM_TRX_DEC_OTHERS  := V_STG_AUTH_STATISTICS_DC_REC.VAL_ATM_TRX_DEC_OTHERS + V_M_STAT_AUTH_ACTIVITY_REC.TRANSACTION_AMOUNT_UC; --MBH10012018 --V_M_STAT_AUTH_ACTIVITY_REC.TRANSACTION_AMOUNT_LC;
            END IF;
            V_POS_ENTRY_MODE := '';
            IF ( SUBSTR(V_M_STAT_AUTH_ACTIVITY_REC.POS_DATA,7,1) = '0' )
            THEN
                V_POS_ENTRY_MODE := 'U' ; --UNSPECIFIED
            ELSIF (    SUBSTR(V_M_STAT_AUTH_ACTIVITY_REC.POS_DATA,7,1) = '1'
                    OR SUBSTR(V_M_STAT_AUTH_ACTIVITY_REC.POS_DATA,7,1) = '6')
            THEN
                V_POS_ENTRY_MODE := 'M';--MANUAL
            ELSIF (    SUBSTR(V_M_STAT_AUTH_ACTIVITY_REC.POS_DATA,7,1) = '2'
                    OR SUBSTR(V_M_STAT_AUTH_ACTIVITY_REC.POS_DATA,7,1) = '3'
                    OR SUBSTR(V_M_STAT_AUTH_ACTIVITY_REC.POS_DATA,7,1) = '4'
                    OR SUBSTR(V_M_STAT_AUTH_ACTIVITY_REC.POS_DATA,7,1) = '7'
                    OR SUBSTR(V_M_STAT_AUTH_ACTIVITY_REC.POS_DATA,7,1) = '8'
                    OR SUBSTR(V_M_STAT_AUTH_ACTIVITY_REC.POS_DATA,7,1) = 'W')
            THEN
                V_POS_ENTRY_MODE := 'S';--MAGNETIC STRIPE
            ELSIF (    SUBSTR(V_M_STAT_AUTH_ACTIVITY_REC.POS_DATA,7,1) = '5'
                    OR SUBSTR(V_M_STAT_AUTH_ACTIVITY_REC.POS_DATA,7,1) = 'J')
            THEN
                V_POS_ENTRY_MODE := 'C';--CHIP
            ELSIF (    SUBSTR(V_M_STAT_AUTH_ACTIVITY_REC.POS_DATA,7,1) = '9'
                    OR SUBSTR(V_M_STAT_AUTH_ACTIVITY_REC.POS_DATA,7,1) = 'S'
                    OR SUBSTR(V_M_STAT_AUTH_ACTIVITY_REC.POS_DATA,7,1) = 'T'
                    OR SUBSTR(V_M_STAT_AUTH_ACTIVITY_REC.POS_DATA,7,1) = 'U'
                    OR SUBSTR(V_M_STAT_AUTH_ACTIVITY_REC.POS_DATA,7,1) = 'V')
            THEN
                V_POS_ENTRY_MODE := 'E';--E-COMMERCE
            ELSE
                V_POS_ENTRY_MODE := '';
            END IF;
            IF (V_M_STAT_AUTH_ACTIVITY_REC.ACTION_CODE = '116'  AND SUBSTR(V_M_STAT_AUTH_ACTIVITY_REC.PROCESSING_CODE,1,2) = '00')
            THEN
                IF (NVL(V_POS_ENTRY_MODE,'X') = 'C')
                THEN
                    V_STG_AUTH_STATISTICS_DC_REC.NBR_P_CHIP_TRX_DEC_INS_FUND  := V_STG_AUTH_STATISTICS_DC_REC.NBR_P_CHIP_TRX_DEC_INS_FUND + V_M_STAT_AUTH_ACTIVITY_REC.NB_AUTHORIZATION;
                    V_STG_AUTH_STATISTICS_DC_REC.VAL_P_CHIP_TRX_DEC_INS_FUND  := V_STG_AUTH_STATISTICS_DC_REC.VAL_P_CHIP_TRX_DEC_INS_FUND + V_M_STAT_AUTH_ACTIVITY_REC.TRANSACTION_AMOUNT_UC; --MBH10012018 --V_M_STAT_AUTH_ACTIVITY_REC.TRANSACTION_AMOUNT_LC;
                ELSIF (NVL(V_POS_ENTRY_MODE,'X') = 'E')
                THEN
                    V_STG_AUTH_STATISTICS_DC_REC.NBR_P_ECOM_TRX_DEC_INS_FUND  := V_STG_AUTH_STATISTICS_DC_REC.NBR_P_ECOM_TRX_DEC_INS_FUND + V_M_STAT_AUTH_ACTIVITY_REC.NB_AUTHORIZATION;
                    V_STG_AUTH_STATISTICS_DC_REC.VAL_P_ECOM_TRX_DEC_INS_FUND  := V_STG_AUTH_STATISTICS_DC_REC.VAL_P_ECOM_TRX_DEC_INS_FUND + V_M_STAT_AUTH_ACTIVITY_REC.TRANSACTION_AMOUNT_UC; --MBH10012018 --V_M_STAT_AUTH_ACTIVITY_REC.TRANSACTION_AMOUNT_LC;
                ELSE
                    V_STG_AUTH_STATISTICS_DC_REC.NBR_PURCH_TRX_DEC_INS_FUND  := V_STG_AUTH_STATISTICS_DC_REC.NBR_PURCH_TRX_DEC_INS_FUND + V_M_STAT_AUTH_ACTIVITY_REC.NB_AUTHORIZATION;
                    V_STG_AUTH_STATISTICS_DC_REC.VAL_PURCH_TRX_DEC_INS_FUND  := V_STG_AUTH_STATISTICS_DC_REC.VAL_PURCH_TRX_DEC_INS_FUND + V_M_STAT_AUTH_ACTIVITY_REC.TRANSACTION_AMOUNT_UC; --MBH10012018 --V_M_STAT_AUTH_ACTIVITY_REC.TRANSACTION_AMOUNT_LC;
                END IF;
            END IF;
            IF (V_ACTION_LIST_RECORD.ACTION_FLAG = 'P'  AND SUBSTR(V_M_STAT_AUTH_ACTIVITY_REC.PROCESSING_CODE,1,2) = '00')
            THEN
                IF (NVL(V_POS_ENTRY_MODE,'X') = 'C')
                THEN
                    V_STG_AUTH_STATISTICS_DC_REC.NBR_P_CHIP_TRX_DEC_INS_FUND  := V_STG_AUTH_STATISTICS_DC_REC.NBR_P_CHIP_TRX_DEC_INS_FUND + V_M_STAT_AUTH_ACTIVITY_REC.NB_AUTHORIZATION;
                    V_STG_AUTH_STATISTICS_DC_REC.VAL_P_CHIP_TRX_DEC_INS_FUND  := V_STG_AUTH_STATISTICS_DC_REC.VAL_P_CHIP_TRX_DEC_INS_FUND + V_M_STAT_AUTH_ACTIVITY_REC.TRANSACTION_AMOUNT_UC; --MBH10012018 --V_M_STAT_AUTH_ACTIVITY_REC.TRANSACTION_AMOUNT_LC;
                ELSIF (NVL(V_POS_ENTRY_MODE,'X') = 'E')
                THEN
                    V_STG_AUTH_STATISTICS_DC_REC.NBR_P_ECOM_TRX_DEC_PICKUP  := V_STG_AUTH_STATISTICS_DC_REC.NBR_P_ECOM_TRX_DEC_PICKUP + V_M_STAT_AUTH_ACTIVITY_REC.NB_AUTHORIZATION;
                    V_STG_AUTH_STATISTICS_DC_REC.VAL_P_ECOM_TRX_DEC_PICKUP  := V_STG_AUTH_STATISTICS_DC_REC.VAL_P_ECOM_TRX_DEC_PICKUP + V_M_STAT_AUTH_ACTIVITY_REC.TRANSACTION_AMOUNT_UC; --MBH10012018 --V_M_STAT_AUTH_ACTIVITY_REC.TRANSACTION_AMOUNT_LC;
                ELSE
                    V_STG_AUTH_STATISTICS_DC_REC.NBR_PURCH_TRX_DEC_PICKUP  := V_STG_AUTH_STATISTICS_DC_REC.NBR_PURCH_TRX_DEC_PICKUP + V_M_STAT_AUTH_ACTIVITY_REC.NB_AUTHORIZATION;
                    V_STG_AUTH_STATISTICS_DC_REC.VAL_PURCH_TRX_DEC_PICKUP  := V_STG_AUTH_STATISTICS_DC_REC.VAL_PURCH_TRX_DEC_PICKUP + V_M_STAT_AUTH_ACTIVITY_REC.TRANSACTION_AMOUNT_UC; --MBH10012018 --V_M_STAT_AUTH_ACTIVITY_REC.TRANSACTION_AMOUNT_LC;
                END IF;
            END IF;
            IF (V_ACTION_LIST_RECORD.ACTION_FLAG <> 'A'  AND SUBSTR(V_M_STAT_AUTH_ACTIVITY_REC.PROCESSING_CODE,1,2) = '00' AND V_M_STAT_AUTH_ACTIVITY_REC.ACTION_CODE <> '116')
            THEN
                IF (NVL(V_POS_ENTRY_MODE,'X') = 'C')
                THEN
                    V_STG_AUTH_STATISTICS_DC_REC.NBR_P_CHIP_TRX_DEC_OTHERS  := V_STG_AUTH_STATISTICS_DC_REC.NBR_P_CHIP_TRX_DEC_OTHERS + V_M_STAT_AUTH_ACTIVITY_REC.NB_AUTHORIZATION;
                    V_STG_AUTH_STATISTICS_DC_REC.VAL_P_CHIP_TRX_DEC_OTHERS  := V_STG_AUTH_STATISTICS_DC_REC.VAL_P_CHIP_TRX_DEC_OTHERS + V_M_STAT_AUTH_ACTIVITY_REC.TRANSACTION_AMOUNT_UC; --MBH10012018 --V_M_STAT_AUTH_ACTIVITY_REC.TRANSACTION_AMOUNT_LC;
                ELSIF (NVL(V_POS_ENTRY_MODE,'X') = 'E')
                THEN
                    V_STG_AUTH_STATISTICS_DC_REC.NBR_P_ECOM_TRX_DEC_OTHERS  := V_STG_AUTH_STATISTICS_DC_REC.NBR_P_ECOM_TRX_DEC_OTHERS+ V_M_STAT_AUTH_ACTIVITY_REC.NB_AUTHORIZATION;
                    V_STG_AUTH_STATISTICS_DC_REC.VAL_P_ECOM_TRX_DEC_OTHERS  := V_STG_AUTH_STATISTICS_DC_REC.VAL_P_ECOM_TRX_DEC_OTHERS + V_M_STAT_AUTH_ACTIVITY_REC.TRANSACTION_AMOUNT_UC; --MBH10012018 --V_M_STAT_AUTH_ACTIVITY_REC.TRANSACTION_AMOUNT_LC;
                ELSE
                    V_STG_AUTH_STATISTICS_DC_REC.NBR_PURCH_TRX_DEC_OTHERS  := V_STG_AUTH_STATISTICS_DC_REC.NBR_PURCH_TRX_DEC_OTHERS + V_M_STAT_AUTH_ACTIVITY_REC.NB_AUTHORIZATION;
                    V_STG_AUTH_STATISTICS_DC_REC.VAL_PURCH_TRX_DEC_OTHERS  := V_STG_AUTH_STATISTICS_DC_REC.VAL_PURCH_TRX_DEC_OTHERS + V_M_STAT_AUTH_ACTIVITY_REC.TRANSACTION_AMOUNT_UC; --MBH10012018 --V_M_STAT_AUTH_ACTIVITY_REC.TRANSACTION_AMOUNT_LC;
                END IF;
            END IF;
        END LOOP;
        IF (V_STG_AUTH_STATISTICS_DC_REC.LOCAL_CURRENCY_CODE IS NULL)
        THEN
            RETURN_STATUS   :=  PCRD_GET_PARAM_GENERAL_ROWS.GET_CURRENCY_TABLE (P_BANK_RECORD.BANK_CURRENCY_CODE,
                                                                                V_CURRENCY_TABLE_RECORD,
                                                                                TRUE);
            IF RETURN_STATUS <> DECLARATION_CST.OK
            THEN
                    V_ENV_INFO_TRACE.USER_MESSAGE := ':ERROR RETURNED BY PCRD_GET_PARAM_GENERAL_ROWS.GET_CURRENCY_TABLE';
                    PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
                    RETURN(RETURN_STATUS);
            END IF;
            V_STG_AUTH_STATISTICS_DC_REC.LOCAL_CURRENCY_CODE     :=  V_CURRENCY_TABLE_RECORD.CURRENCY_CODE;
            V_STG_AUTH_STATISTICS_DC_REC.LOCAL_CURRENCY_EXPONENT :=  V_CURRENCY_TABLE_RECORD.CURRENCY_EXPONENT;
        END IF;
        RETURN_STATUS   :=  PCRD_STA_REPORTING_TOOLS_1_V3.PUT_ON_STG_AUTH_STATISTICS_DC(V_STG_AUTH_STATISTICS_DC_REC);
        IF (RETURN_STATUS <> DECLARATION_CST.OK)
        THEN
            V_ENV_INFO_TRACE.USER_MESSAGE := 'ERROR RETURNED BY PCRD_STA_REPORTING_TOOLS_1_V3.PUT_ON_STG_AUTH_STATISTICS_DC STATISTICS_PERIOD = ' ||  V_STG_AUTH_STATISTICS_DC_REC.STATISTICS_PERIOD ||
                                ', BANK_CODE = ' || V_STG_AUTH_STATISTICS_DC_REC.BANK_CODE ||
                                ', NETWORK_CODE = ' || V_STG_AUTH_STATISTICS_DC_REC.NETWORK_CODE ||
                                ', CLASS_GROUP = ' || V_STG_AUTH_STATISTICS_DC_REC.CLASS_GROUP ||
                                 ', CLASS = ' || V_STG_AUTH_STATISTICS_DC_REC.CLASS ;
            PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
            RETURN (DECLARATION_CST.ERROR);
        END IF;
        EXIT WHEN V_COMPTEUR = 3;
        V_COMPTEUR := V_COMPTEUR + 1;
        V_START_DATE    :=  ADD_MONTHS(V_START_DATE,1);
    END LOOP;
    RETURN(DECLARATION_CST.OK);
EXCEPTION WHEN OTHERS
THEN
    V_ENV_INFO_TRACE.USER_MESSAGE   :=  NULL;
    PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
    RETURN(DECLARATION_CST.NOK);
END PREP_STG_AUTH_STATISTICS_DC;
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
FUNCTION        PUT_ON_STG_AUTH_STATISTICS_DC  (P_STG_AUTH_STATISTICS_DC_REC  IN           STG_AUTH_STATISTICS_DC%ROWTYPE)
                                                RETURN PLS_INTEGER IS
V_ENV_INFO_TRACE              GLOBAL_VARS.ENV_INFO_TRACE_TYPE:= NULL;
BEGIN
        V_ENV_INFO_TRACE.USER_NAME      :=  GLOBAL_VARS.USER_NAME;
        V_ENV_INFO_TRACE.MODULE_CODE    :=  GLOBAL_VARS.ML_STATISTICS;
        V_ENV_INFO_TRACE.PACKAGE_NAME   :=  $$PLSQL_UNIT;
        V_ENV_INFO_TRACE.FUNCTION_NAME  :=  'PUT_ON_STG_AUTH_STATISTICS_DC';
        V_ENV_INFO_TRACE.LANG           :=  GLOBAL_VARS.LANG;

        INSERT INTO STG_AUTH_STATISTICS_DC VALUES  P_STG_AUTH_STATISTICS_DC_REC;
        RETURN(DECLARATION_CST.OK);
EXCEPTION
WHEN    OTHERS  THEN
        V_ENV_INFO_TRACE.ERROR_CODE   := 'PWC-00016';
        V_ENV_INFO_TRACE.PARAM1       := 'STG_AUTH_STATISTICS_CR';
        V_ENV_INFO_TRACE.PARAM2       := 'STATISTICS_PERIOD = ' ||  P_STG_AUTH_STATISTICS_DC_REC.STATISTICS_PERIOD ||
                                        ', BANK_CODE = ' || P_STG_AUTH_STATISTICS_DC_REC.BANK_CODE ||
                                        ', NETWORK_CODE = ' || P_STG_AUTH_STATISTICS_DC_REC.NETWORK_CODE ||
                                        ', CLASS_GROUP = ' || P_STG_AUTH_STATISTICS_DC_REC.CLASS_GROUP ||
                                         ', CLASS = ' || P_STG_AUTH_STATISTICS_DC_REC.CLASS ;
        V_ENV_INFO_TRACE.USER_MESSAGE :=  NULL;
        PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
        RETURN( DECLARATION_CST.ERROR );
END PUT_ON_STG_AUTH_STATISTICS_DC;
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
FUNCTION   PREP_STG_ACCOUNT_DELINQ_DWH_CR(  P_BUSINESS_DATE                     IN          DATE)
                                            RETURN PLS_INTEGER IS

RETURN_STATUS                           PLS_INTEGER;
V_START_DATE                            DATE;
V_COMPTEUR                              PLS_INTEGER;
V_STG_ACCOUNT_DELINQ_CR_REC             STG_ACCOUNT_DELINQUENCIES_CR%ROWTYPE;
V_AVAILABLE_BALANCE                     SHADOW_ACCOUNT.CLOSING_BALANCE_PURCHASE%TYPE;
V_GLOBAL_BALANCE                        SHADOW_ACCOUNT.CLOSING_BALANCE_PURCHASE%TYPE;
V_GLOBAL_PRINCIPAL_BALANCE              SHADOW_ACCOUNT.CLOSING_BALANCE_PURCHASE%TYPE;
V_CR_TERM_RECORD                        CR_TERM%ROWTYPE;
V_DAYS_AFTER_DUEDATE                    NUMBER;
V_CURRENCY_TABLE_RECORD                 CURRENCY_TABLE%ROWTYPE ;
V_TOT_CREDIT_LIMIT_LC                   SHADOW_ACCOUNT.CREDIT_LIMIT%TYPE;
V_CREDIT_LIMIT_LC                       SHADOW_ACCOUNT.CREDIT_LIMIT%TYPE;
V_CONVERSION_DATE                       CONVERSION_RATE.RATE_DATE%TYPE;
V_RATE_ISO_STR_FORMAT                   VARCHAR2(9);
V_MARKUP_AMOUNT                         TRANSACTION_HIST.CONVERSION_FEES%TYPE;
V_RATE_DATE                             DATE;
V_REPORT_CODE                           PWC_REPORT_PARAMETERS.REPORT_CODE%TYPE:= 'STA_01_LAC_CREDIT_QTR';
V_BANK_RECORD                           BANK%ROWTYPE;
V_CUTOFF_SEQUENCE                       CUTOFF_FOLLOW_UP.CUTOFF_SEQUENCE%TYPE;
V_NETWORK_RECORD                        NETWORK%ROWTYPE;
V_KEY_VAL                               RESSOURCE_BUNDLE.KEY_VAL%TYPE;
V_BUNDLE_REPORT_V3                      BUNDLE_REPORT_V3;

CURSOR  CUR_DWH_ACC_DAILY_SUMMARY (P_BANK_CODE BANK.BANK_CODE%TYPE,P_CLASS  CARD_TYPE.CLASS%TYPE,P_CLASS_GROUP CARD_TYPE.CLASS_GROUP%TYPE)
IS
SELECT  A.*
FROM DWH_ACC_DAILY_SUMMARY A, CARD_PRODUCT B, CARD_TYPE C
WHERE A.BANK_CODE           = B.BANK_CODE
AND   B.BANK_CODE           = C.BANK_CODE
AND   A.PRODUCT_CODE        = B.PRODUCT_CODE
AND   B.NETWORK_CARD_TYPE   = C.NETWORK_CARD_TYPE
AND   C.CLASS               = P_CLASS
AND   C.CLASS_GROUP         = P_CLASS_GROUP
AND   A.BANK_CODE           = P_BANK_CODE
AND   B.NETWORK_CODE        = GLOBAL_VARS.NETWORK_VISA
AND   ACCOUNT_STATUS_CODE   <> '6'
AND   OUTSTANDING_AMOUNT    <> 0
AND   PRODUCT_TYPE          = GLOBAL_VARS.CREDIT_CARD_TYPE;

CURSOR  CUR_CARD_TYPE (P_BANK_CODE BANK.BANK_CODE%TYPE)  IS
SELECT  COUNT(1),BANK_CODE,NETWORK_CODE,CLASS,CLASS_GROUP
FROM    CARD_TYPE
WHERE   BANK_CODE = P_BANK_CODE
AND NETWORK_CODE = GLOBAL_VARS.NETWORK_VISA
AND CLASS_LEVEL  = '2' --SAB16072015
GROUP BY BANK_CODE,NETWORK_CODE,CLASS,CLASS_GROUP;

CURSOR CUR_PWC_BANK_REPORT_PARAMETERS   IS
    SELECT *
    FROM PWC_BANK_REPORT_PARAMETERS
    WHERE REPORT_CODE = V_REPORT_CODE;

BEGIN
          V_ENV_INFO_TRACE.USER_NAME      :=  GLOBAL_VARS.USER_NAME;
          V_ENV_INFO_TRACE.MODULE_CODE    :=  GLOBAL_VARS.ML_STATISTICS;
          V_ENV_INFO_TRACE.LANG           :=  GLOBAL_VARS.LANG;
          V_ENV_INFO_TRACE.PACKAGE_NAME   :=  $$PLSQL_UNIT;
          V_ENV_INFO_TRACE.FUNCTION_NAME  :=  'PREP_STG_ACCOUNT_DELINQ_DWH_CR';

    IF (TRUNC(P_BUSINESS_DATE) = LAST_DAY(TO_DATE(TO_CHAR(P_BUSINESS_DATE,'DD')||'/'||TO_CHAR(P_BUSINESS_DATE,'MM')||'/'||TO_CHAR(P_BUSINESS_DATE,'YYYY'),'DD/MM/YYYY')))
    THEN

        FOR V_PWC_BANK_REPORT_PARAM_REC IN CUR_PWC_BANK_REPORT_PARAMETERS
        LOOP
            --START MBH21042017
            DELETE FROM  STG_ACCOUNT_DELINQUENCIES_CR
            WHERE  STATISTICS_PERIOD  = TRUNC(P_BUSINESS_DATE,'MONTH')
            AND BANK_CODE = V_PWC_BANK_REPORT_PARAM_REC.BANK_CODE;
            --END MBH21042017
            RETURN_STATUS := PCRD_GET_PARAM_GENERAL_ROWS.GET_BANK ( V_PWC_BANK_REPORT_PARAM_REC.BANK_CODE,
                                                                    V_BANK_RECORD );
            IF RETURN_STATUS <> DECLARATION_CST.OK
            THEN
                V_ENV_INFO_TRACE.USER_MESSAGE   :=  'ERROR RETURNED BY PCRD_GET_PARAM_GENERAL_ROWS.GET_BANK'
                                                ||  ', BANK_CODE : '         || V_PWC_BANK_REPORT_PARAM_REC.BANK_CODE;
                PCRD_GENERAL_TOOLS.PUT_TRACES  (V_ENV_INFO_TRACE, $$PLSQL_LINE  );
                RETURN  RETURN_STATUS;
            END IF;

            RETURN_STATUS   :=  PCRD_GET_PARAM_GENERAL_ROWS.GET_NETWORK  (  GLOBAL_VARS.NETWORK_VISA,
                                                                            V_NETWORK_RECORD);
            IF RETURN_STATUS <> DECLARATION_CST.OK
            THEN
                V_ENV_INFO_TRACE.USER_MESSAGE := ':ERROR RETURNED BY PCRD_GET_PARAM_GENERAL_ROWS.GET_NETWORK '
                                                || 'NETWORK CODE :' ||GLOBAL_VARS.NETWORK_VISA;
                PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
                RETURN (RETURN_STATUS);
            END IF;

            RETURN_STATUS   :=  PCRD_GET_PARAM_GENERAL_ROWS.GET_CURRENCY_TABLE  (   V_BANK_RECORD.BANK_CURRENCY_CODE,
                                                                                    V_CURRENCY_TABLE_RECORD,
                                                                                    TRUE);
            IF RETURN_STATUS <> DECLARATION_CST.OK
            THEN
                    V_ENV_INFO_TRACE.USER_MESSAGE := ':ERROR RETURNED BY PCRD_GET_PARAM_GENERAL_ROWS.GET_CURRENCY_TABLE';
                    PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
                    RETURN(RETURN_STATUS);
            END IF;
            RETURN_STATUS   :=  PCRD_CUTOFF_FOLLOW_UP.READ_CUTOFF_SEQUENCE( P_BUSINESS_DATE,
                                                                            GLOBAL_VARS_CAI_1.STATISTICS,
                                                                            NULL,
                                                                            V_CUTOFF_SEQUENCE   );
            IF  RETURN_STATUS   <>  DECLARATION_CST.OK
            THEN
                V_ENV_INFO_TRACE.USER_MESSAGE :=  'ERROR RETURNED BY PCRD_CUTOFF_FOLLOW_UP.READ_CUTOFF_SEQUENCE';
                PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
                RETURN RETURN_STATUS;
            END IF;
            FOR V_CUR_CARD_TYPE_REC IN CUR_CARD_TYPE (V_PWC_BANK_REPORT_PARAM_REC.BANK_CODE)
            LOOP
                V_STG_ACCOUNT_DELINQ_CR_REC                                :=  NULL;
                V_STG_ACCOUNT_DELINQ_CR_REC.BUSINESS_DATE                  :=  P_BUSINESS_DATE;
                V_STG_ACCOUNT_DELINQ_CR_REC.BANK_CODE                      :=  V_BANK_RECORD.BANK_CODE;
                V_STG_ACCOUNT_DELINQ_CR_REC.BANK_NAME                      :=  V_BANK_RECORD.BANK_NAME;
                V_STG_ACCOUNT_DELINQ_CR_REC.NETWORK_CODE                   :=  V_NETWORK_RECORD.NETWORK_CODE;
                V_STG_ACCOUNT_DELINQ_CR_REC.NETWORK_NAME                   :=  V_NETWORK_RECORD.NETWORK_LABEL;
                V_STG_ACCOUNT_DELINQ_CR_REC.CUTOFF_ID                      :=  V_CUTOFF_SEQUENCE;
                V_STG_ACCOUNT_DELINQ_CR_REC.LANGUAGE_1                     :=  V_PWC_BANK_REPORT_PARAM_REC.LANGUAGE_1;
                V_STG_ACCOUNT_DELINQ_CR_REC.LANGUAGE_2                     :=  V_PWC_BANK_REPORT_PARAM_REC.LANGUAGE_2;
                V_STG_ACCOUNT_DELINQ_CR_REC.LANGUAGE_3                     :=  V_PWC_BANK_REPORT_PARAM_REC.LANGUAGE_3;
                V_STG_ACCOUNT_DELINQ_CR_REC.CURRENCY_CODE                  :=  V_CURRENCY_TABLE_RECORD.CURRENCY_CODE;
                V_STG_ACCOUNT_DELINQ_CR_REC.CURRENCY_EXPONENT              :=  V_CURRENCY_TABLE_RECORD.CURRENCY_EXPONENT;
                V_STG_ACCOUNT_DELINQ_CR_REC.STATISTICS_PERIOD              :=  TRUNC(P_BUSINESS_DATE,'MONTH');
                V_STG_ACCOUNT_DELINQ_CR_REC.CLASS                          :=  V_CUR_CARD_TYPE_REC.CLASS;
                V_STG_ACCOUNT_DELINQ_CR_REC.CLASS_GROUP                    :=  V_CUR_CARD_TYPE_REC.CLASS_GROUP;
                V_KEY_VAL := '';
                CASE    V_STG_ACCOUNT_DELINQ_CR_REC.CLASS_GROUP
                WHEN    'IN'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS_GROUP.IN';
                WHEN    'BU'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS_GROUP.BU';
                ELSE    V_KEY_VAL := 'NA';
                END CASE;
                V_BUNDLE_REPORT_V3 := PCRD_REPORTING_TOOLS_V3.NEW_BUNDLE_REPORT_V3;
                V_BUNDLE_REPORT_V3.KEY_VAL                      := V_KEY_VAL;
                V_BUNDLE_REPORT_V3.LANGUAGE_1                   := V_STG_ACCOUNT_DELINQ_CR_REC.LANGUAGE_1;
                V_BUNDLE_REPORT_V3.LANGUAGE_2                   := V_STG_ACCOUNT_DELINQ_CR_REC.LANGUAGE_2;
                V_BUNDLE_REPORT_V3.LANGUAGE_3                   := V_STG_ACCOUNT_DELINQ_CR_REC.LANGUAGE_3;
                RETURN_STATUS   :=  PCRD_REPORTING_TOOLS_V3.GET_REPORT_LABELS   ( V_BUNDLE_REPORT_V3 );
                IF RETURN_STATUS <> DECLARATION_CST.OK
                THEN
                    V_ENV_INFO_TRACE.USER_MESSAGE   :=  'ERROR RETURNED BY PCRD_REPORTING_TOOLS_V3.GET_REPORT_LABELS'  ||
                                                        ',BUNDLE : ' || PCRD_REPORTING_TOOLS_V3.BUNDLE ||
                                                        ',KEY_VAL : ' || V_KEY_VAL;
                    PCRD_GENERAL_TOOLS.PUT_TRACES  (V_ENV_INFO_TRACE, $$PLSQL_LINE  );
                    RETURN  ( DECLARATION_CST.ERROR );
                END IF;
                V_STG_ACCOUNT_DELINQ_CR_REC.CLASS_GROUP_DESC_1             :=  V_BUNDLE_REPORT_V3.VALUE_1;
                V_STG_ACCOUNT_DELINQ_CR_REC.CLASS_GROUP_DESC_2             :=  V_BUNDLE_REPORT_V3.VALUE_2;
                V_STG_ACCOUNT_DELINQ_CR_REC.CLASS_GROUP_DESC_3             :=  V_BUNDLE_REPORT_V3.VALUE_3;
                V_KEY_VAL := '';
                CASE    V_STG_ACCOUNT_DELINQ_CR_REC.CLASS
                WHEN    '01'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS.1';
                WHEN    '02'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS.2';
                WHEN    '03'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS.3';
                WHEN    '04'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS.4';
                WHEN    '05'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS.5';
                WHEN    '06'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS.6';
                ELSE    V_KEY_VAL := 'NA';
                END CASE;
                V_BUNDLE_REPORT_V3 := PCRD_REPORTING_TOOLS_V3.NEW_BUNDLE_REPORT_V3;
                V_BUNDLE_REPORT_V3.KEY_VAL                      := V_KEY_VAL;
                V_BUNDLE_REPORT_V3.LANGUAGE_1                   := V_STG_ACCOUNT_DELINQ_CR_REC.LANGUAGE_1;
                V_BUNDLE_REPORT_V3.LANGUAGE_2                   := V_STG_ACCOUNT_DELINQ_CR_REC.LANGUAGE_2;
                V_BUNDLE_REPORT_V3.LANGUAGE_3                   := V_STG_ACCOUNT_DELINQ_CR_REC.LANGUAGE_3;
                RETURN_STATUS   :=  PCRD_REPORTING_TOOLS_V3.GET_REPORT_LABELS   ( V_BUNDLE_REPORT_V3 );
                IF RETURN_STATUS <> DECLARATION_CST.OK
                THEN
                    V_ENV_INFO_TRACE.USER_MESSAGE   :=  'ERROR RETURNED BY PCRD_REPORTING_TOOLS_V3.GET_REPORT_LABELS'  ||
                                                        ',BUNDLE : ' || PCRD_REPORTING_TOOLS_V3.BUNDLE ||
                                                        ',KEY_VAL : ' || V_KEY_VAL;
                    PCRD_GENERAL_TOOLS.PUT_TRACES  (V_ENV_INFO_TRACE, $$PLSQL_LINE  );
                    RETURN  ( DECLARATION_CST.ERROR );
                END IF;
                V_STG_ACCOUNT_DELINQ_CR_REC.CLASS_DESCRIPTION_1            :=  V_BUNDLE_REPORT_V3.VALUE_1;
                V_STG_ACCOUNT_DELINQ_CR_REC.CLASS_DESCRIPTION_2            :=  V_BUNDLE_REPORT_V3.VALUE_2;
                V_STG_ACCOUNT_DELINQ_CR_REC.CLASS_DESCRIPTION_3            :=  V_BUNDLE_REPORT_V3.VALUE_3;
                V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQ_ACC_LESS_30_DAYS    :=  0.;
                V_STG_ACCOUNT_DELINQ_CR_REC.VAL_DELINQ_ACC_LESS_30_DAYS    :=  0.;
                V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQ_ACC_OVER_120_DAYS   :=  0.;
                V_STG_ACCOUNT_DELINQ_CR_REC.VAL_DELINQ_ACC_OVER_120_DAYS   :=  0.;
                V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQUENT_ACC_30_60_DAYS  :=  0.;
                V_STG_ACCOUNT_DELINQ_CR_REC.VAL_DELINQUENT_ACC_30_60_DAYS  :=  0.;
                V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQUENT_ACC_60_90_DAYS  :=  0.;
                V_STG_ACCOUNT_DELINQ_CR_REC.VAL_DELINQUENT_ACC_60_90_DAYS  :=  0.;
                V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQUENT_ACC_90_120_DAYS :=  0.;
                V_STG_ACCOUNT_DELINQ_CR_REC.VAL_DELINQUENT_ACC_90_120_DAYS :=  0.;
                V_STG_ACCOUNT_DELINQ_CR_REC.TOTAL_NBR_DELINQUENT_ACCOUNTS  :=  0.;
                V_STG_ACCOUNT_DELINQ_CR_REC.TOTAL_VAL_DELINQUENT_ACCOUNTS  :=  0.;
                V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQ_ACC_WITH_GRACE_PER  :=  0.;
                V_STG_ACCOUNT_DELINQ_CR_REC.VAL_DELINQ_ACC_WITH_GRACE_PER  :=  0.;
                V_STG_ACCOUNT_DELINQ_CR_REC.TOT_OF_CREDIT_LIMIT_LC         :=  0.;
                V_STG_ACCOUNT_DELINQ_CR_REC.TOT_OF_CREDIT_LIMIT_UC         :=  0.;
                FOR V_CUR_DWH_ACC_DAILY_SUM_REC IN CUR_DWH_ACC_DAILY_SUMMARY (V_PWC_BANK_REPORT_PARAM_REC.BANK_CODE,V_STG_ACCOUNT_DELINQ_CR_REC.CLASS,V_STG_ACCOUNT_DELINQ_CR_REC.CLASS_GROUP)
                LOOP
                    CASE V_CUR_DWH_ACC_DAILY_SUM_REC.DPD_CLASSIFICATION_CODE
                    WHEN '0' THEN V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQ_ACC_LESS_30_DAYS   := V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQ_ACC_LESS_30_DAYS + 1;
                                  V_STG_ACCOUNT_DELINQ_CR_REC.VAL_DELINQ_ACC_LESS_30_DAYS   := V_STG_ACCOUNT_DELINQ_CR_REC.VAL_DELINQ_ACC_LESS_30_DAYS + NVL(V_CUR_DWH_ACC_DAILY_SUM_REC.OUTSTANDING_AMOUNT,0);
                    WHEN '1' THEN V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQ_ACC_LESS_30_DAYS   := V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQ_ACC_LESS_30_DAYS + 1;
                                  V_STG_ACCOUNT_DELINQ_CR_REC.VAL_DELINQ_ACC_LESS_30_DAYS   := V_STG_ACCOUNT_DELINQ_CR_REC.VAL_DELINQ_ACC_LESS_30_DAYS + NVL(V_CUR_DWH_ACC_DAILY_SUM_REC.OUTSTANDING_AMOUNT,0);
                    WHEN '2' THEN V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQUENT_ACC_30_60_DAYS := V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQUENT_ACC_30_60_DAYS + 1;
                                  V_STG_ACCOUNT_DELINQ_CR_REC.VAL_DELINQUENT_ACC_30_60_DAYS := V_STG_ACCOUNT_DELINQ_CR_REC.VAL_DELINQUENT_ACC_30_60_DAYS + NVL(V_CUR_DWH_ACC_DAILY_SUM_REC.OUTSTANDING_AMOUNT,0);
                    WHEN '3' THEN V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQUENT_ACC_60_90_DAYS := V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQUENT_ACC_60_90_DAYS + 1;
                                  V_STG_ACCOUNT_DELINQ_CR_REC.VAL_DELINQUENT_ACC_60_90_DAYS := V_STG_ACCOUNT_DELINQ_CR_REC.VAL_DELINQUENT_ACC_60_90_DAYS + NVL(V_CUR_DWH_ACC_DAILY_SUM_REC.OUTSTANDING_AMOUNT,0);
                    WHEN '4' THEN V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQUENT_ACC_90_120_DAYS:= V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQUENT_ACC_90_120_DAYS + 1;
                                  V_STG_ACCOUNT_DELINQ_CR_REC.VAL_DELINQUENT_ACC_90_120_DAYS:= V_STG_ACCOUNT_DELINQ_CR_REC.VAL_DELINQUENT_ACC_90_120_DAYS + NVL(V_CUR_DWH_ACC_DAILY_SUM_REC.OUTSTANDING_AMOUNT,0);

                    ELSE
                        V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQ_ACC_OVER_120_DAYS := V_STG_ACCOUNT_DELINQ_CR_REC.NBR_DELINQ_ACC_OVER_120_DAYS + 1;
                        V_STG_ACCOUNT_DELINQ_CR_REC.VAL_DELINQ_ACC_OVER_120_DAYS := V_STG_ACCOUNT_DELINQ_CR_REC.VAL_DELINQ_ACC_OVER_120_DAYS +  NVL(V_CUR_DWH_ACC_DAILY_SUM_REC.OUTSTANDING_AMOUNT,0);
                    END CASE;
                    V_STG_ACCOUNT_DELINQ_CR_REC.TOTAL_NBR_DELINQUENT_ACCOUNTS       := V_STG_ACCOUNT_DELINQ_CR_REC.TOTAL_NBR_DELINQUENT_ACCOUNTS + 1;
                    V_STG_ACCOUNT_DELINQ_CR_REC.TOTAL_VAL_DELINQUENT_ACCOUNTS       := V_STG_ACCOUNT_DELINQ_CR_REC.TOTAL_VAL_DELINQUENT_ACCOUNTS + NVL(V_CUR_DWH_ACC_DAILY_SUM_REC.OUTSTANDING_AMOUNT,0);
                    V_STG_ACCOUNT_DELINQ_CR_REC.TOT_OF_CREDIT_LIMIT_LC              := V_STG_ACCOUNT_DELINQ_CR_REC.TOT_OF_CREDIT_LIMIT_LC + NVL(V_CUR_DWH_ACC_DAILY_SUM_REC.CREDIT_LIMIT_APPROVED_AMOUNT,0);
                END LOOP;
                RETURN_STATUS   :=  PCRD_STA_REPORTING_TOOLS_1_V3.PUT_ON_STG_ACCOUNT_DELINQ_CR(V_STG_ACCOUNT_DELINQ_CR_REC);
                IF (RETURN_STATUS <> DECLARATION_CST.OK)
                THEN
                    V_ENV_INFO_TRACE.USER_MESSAGE := 'ERROR RETURNED BY PCRD_STA_REPORTING_TOOLS_1_V3.PUT_ON_STG_ACCOUNT_DELINQ_CR, STATISTICS_PERIOD = ' ||  V_STG_ACCOUNT_DELINQ_CR_REC.STATISTICS_PERIOD ||
                                        ', BANK_CODE = ' || V_STG_ACCOUNT_DELINQ_CR_REC.BANK_CODE ||
                                        ', STATISTICS_PERIOD = ' || V_STG_ACCOUNT_DELINQ_CR_REC.STATISTICS_PERIOD ||
                                        ', CLASS = ' || V_STG_ACCOUNT_DELINQ_CR_REC.CLASS ||
                                        ', CLASS_GROUP = ' || V_STG_ACCOUNT_DELINQ_CR_REC.CLASS_GROUP ||
                                        ', NETWORK_CODE = ' || V_STG_ACCOUNT_DELINQ_CR_REC.NETWORK_CODE ;
                    PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
                    RETURN (DECLARATION_CST.ERROR);
                END IF;
            END LOOP;
        END LOOP;
    END IF;
    COMMIT;
    RETURN(DECLARATION_CST.OK);
EXCEPTION WHEN OTHERS
THEN
    V_ENV_INFO_TRACE.USER_MESSAGE   :=  NULL;
    PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
    RETURN(DECLARATION_CST.NOK);
END PREP_STG_ACCOUNT_DELINQ_DWH_CR;
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
FUNCTION   PREP_STG_ACCOUNT_DELINQ_DWH_CC(  P_BUSINESS_DATE                     IN          DATE)
                                            RETURN PLS_INTEGER IS

RETURN_STATUS                           PLS_INTEGER;
V_START_DATE                            DATE;
V_COMPTEUR                              PLS_INTEGER;
V_STG_ACCOUNT_DELINQ_CC_REC             STG_ACCOUNT_DELINQUENCIES_CC%ROWTYPE;
V_AVAILABLE_BALANCE                     SHADOW_ACCOUNT.CLOSING_BALANCE_PURCHASE%TYPE;
V_GLOBAL_BALANCE                        SHADOW_ACCOUNT.CLOSING_BALANCE_PURCHASE%TYPE;
V_GLOBAL_PRINCIPAL_BALANCE              SHADOW_ACCOUNT.CLOSING_BALANCE_PURCHASE%TYPE;
V_CR_TERM_RECORD                        CR_TERM%ROWTYPE;
V_DAYS_AFTER_DUEDATE                    NUMBER;
V_CURRENCY_TABLE_RECORD                 CURRENCY_TABLE%ROWTYPE ;
V_TOT_CREDIT_LIMIT_LC                   SHADOW_ACCOUNT.CREDIT_LIMIT%TYPE;
V_CREDIT_LIMIT_LC                       SHADOW_ACCOUNT.CREDIT_LIMIT%TYPE;
V_CONVERSION_DATE                       CONVERSION_RATE.RATE_DATE%TYPE;
V_RATE_ISO_STR_FORMAT                   VARCHAR2(9);
V_MARKUP_AMOUNT                         TRANSACTION_HIST.CONVERSION_FEES%TYPE;
V_RATE_DATE                             DATE;
V_REPORT_CODE                           PWC_REPORT_PARAMETERS.REPORT_CODE%TYPE:= 'STA_24_EUROPE_CHARGE_QTR';
V_BANK_RECORD                           BANK%ROWTYPE;
V_CUTOFF_SEQUENCE                       CUTOFF_FOLLOW_UP.CUTOFF_SEQUENCE%TYPE;
V_NETWORK_RECORD                        NETWORK%ROWTYPE;
V_KEY_VAL                               RESSOURCE_BUNDLE.KEY_VAL%TYPE;
V_BUNDLE_REPORT_V3                      BUNDLE_REPORT_V3;

CURSOR  CUR_DWH_ACC_DAILY_SUMMARY (P_BANK_CODE BANK.BANK_CODE%TYPE,P_CLASS  CARD_TYPE.CLASS%TYPE,P_CLASS_GROUP CARD_TYPE.CLASS_GROUP%TYPE)
IS
SELECT  A.*
FROM DWH_ACC_DAILY_SUMMARY A, CARD_PRODUCT B, CARD_TYPE C
WHERE A.BANK_CODE           = B.BANK_CODE
AND   B.BANK_CODE           = C.BANK_CODE
AND   A.PRODUCT_CODE        = B.PRODUCT_CODE
AND   B.NETWORK_CARD_TYPE   = C.NETWORK_CARD_TYPE
AND   C.CLASS               = P_CLASS
AND   C.CLASS_GROUP         = P_CLASS_GROUP
AND   A.BANK_CODE           = P_BANK_CODE
AND   B.NETWORK_CODE        = GLOBAL_VARS.NETWORK_VISA
AND   ACCOUNT_STATUS_CODE   <> '6'
AND   OUTSTANDING_AMOUNT    <> 0
AND   PRODUCT_TYPE          = GLOBAL_VARS.CHARGE_CARD_TYPE;

CURSOR  CUR_CARD_TYPE (P_BANK_CODE BANK.BANK_CODE%TYPE)  IS
SELECT  COUNT(1),BANK_CODE,NETWORK_CODE,CLASS,CLASS_GROUP
FROM    CARD_TYPE
WHERE   BANK_CODE = P_BANK_CODE
AND NETWORK_CODE  = GLOBAL_VARS.NETWORK_VISA
AND CLASS_LEVEL   = '2'
GROUP BY BANK_CODE,NETWORK_CODE,CLASS,CLASS_GROUP;

CURSOR CUR_PWC_BANK_REPORT_PARAMETERS   IS
    SELECT *
    FROM PWC_BANK_REPORT_PARAMETERS
    WHERE REPORT_CODE = V_REPORT_CODE;

BEGIN
          V_ENV_INFO_TRACE.USER_NAME      :=  GLOBAL_VARS.USER_NAME;
          V_ENV_INFO_TRACE.MODULE_CODE    :=  GLOBAL_VARS.ML_STATISTICS;
          V_ENV_INFO_TRACE.LANG           :=  GLOBAL_VARS.LANG;
          V_ENV_INFO_TRACE.PACKAGE_NAME   :=  $$PLSQL_UNIT;
          V_ENV_INFO_TRACE.FUNCTION_NAME  :=  'PREP_STG_ACCOUNT_DELINQ_DWH_CC';

    IF (TRUNC(P_BUSINESS_DATE) = LAST_DAY(TO_DATE(TO_CHAR(P_BUSINESS_DATE,'DD')||'/'||TO_CHAR(P_BUSINESS_DATE,'MM')||'/'||TO_CHAR(P_BUSINESS_DATE,'YYYY'),'DD/MM/YYYY')))
    THEN

        FOR V_PWC_BANK_REPORT_PARAM_REC IN CUR_PWC_BANK_REPORT_PARAMETERS
        LOOP
            DELETE FROM  STG_ACCOUNT_DELINQUENCIES_CC
            WHERE  STATISTICS_PERIOD  = TRUNC(P_BUSINESS_DATE,'MONTH')
            AND BANK_CODE = V_PWC_BANK_REPORT_PARAM_REC.BANK_CODE;

            RETURN_STATUS := PCRD_GET_PARAM_GENERAL_ROWS.GET_BANK ( V_PWC_BANK_REPORT_PARAM_REC.BANK_CODE,
                                                                    V_BANK_RECORD );
            IF RETURN_STATUS <> DECLARATION_CST.OK
            THEN
                V_ENV_INFO_TRACE.USER_MESSAGE   :=  'ERROR RETURNED BY PCRD_GET_PARAM_GENERAL_ROWS.GET_BANK'
                                                ||  ', BANK_CODE : '         || V_PWC_BANK_REPORT_PARAM_REC.BANK_CODE;
                PCRD_GENERAL_TOOLS.PUT_TRACES  (V_ENV_INFO_TRACE, $$PLSQL_LINE  );
                RETURN  RETURN_STATUS;
            END IF;

            RETURN_STATUS   :=  PCRD_GET_PARAM_GENERAL_ROWS.GET_NETWORK  (  GLOBAL_VARS.NETWORK_VISA,
                                                                            V_NETWORK_RECORD);
            IF RETURN_STATUS <> DECLARATION_CST.OK
            THEN
                V_ENV_INFO_TRACE.USER_MESSAGE := ':ERROR RETURNED BY PCRD_GET_PARAM_GENERAL_ROWS.GET_NETWORK '
                                                || 'NETWORK CODE :' ||GLOBAL_VARS.NETWORK_VISA;
                PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
                RETURN (RETURN_STATUS);
            END IF;

            RETURN_STATUS   :=  PCRD_GET_PARAM_GENERAL_ROWS.GET_CURRENCY_TABLE  (   V_BANK_RECORD.BANK_CURRENCY_CODE,
                                                                                    V_CURRENCY_TABLE_RECORD,
                                                                                    TRUE);
            IF RETURN_STATUS <> DECLARATION_CST.OK
            THEN
                    V_ENV_INFO_TRACE.USER_MESSAGE := ':ERROR RETURNED BY PCRD_GET_PARAM_GENERAL_ROWS.GET_CURRENCY_TABLE';
                    PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
                    RETURN(RETURN_STATUS);
            END IF;
            RETURN_STATUS   :=  PCRD_CUTOFF_FOLLOW_UP.READ_CUTOFF_SEQUENCE( P_BUSINESS_DATE,
                                                                            GLOBAL_VARS_CAI_1.STATISTICS,
                                                                            NULL,
                                                                            V_CUTOFF_SEQUENCE   );
            IF  RETURN_STATUS   <>  DECLARATION_CST.OK
            THEN
                V_ENV_INFO_TRACE.USER_MESSAGE :=  'ERROR RETURNED BY PCRD_CUTOFF_FOLLOW_UP.READ_CUTOFF_SEQUENCE';
                PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
                RETURN RETURN_STATUS;
            END IF;
            FOR V_CUR_CARD_TYPE_REC IN CUR_CARD_TYPE (V_PWC_BANK_REPORT_PARAM_REC.BANK_CODE)
            LOOP
                V_STG_ACCOUNT_DELINQ_CC_REC                                :=  NULL;
                V_STG_ACCOUNT_DELINQ_CC_REC.BUSINESS_DATE                  :=  P_BUSINESS_DATE;
                V_STG_ACCOUNT_DELINQ_CC_REC.BANK_CODE                      :=  V_BANK_RECORD.BANK_CODE;
                V_STG_ACCOUNT_DELINQ_CC_REC.BANK_NAME                      :=  V_BANK_RECORD.BANK_NAME;
                V_STG_ACCOUNT_DELINQ_CC_REC.NETWORK_CODE                   :=  V_NETWORK_RECORD.NETWORK_CODE;
                V_STG_ACCOUNT_DELINQ_CC_REC.NETWORK_NAME                   :=  V_NETWORK_RECORD.NETWORK_LABEL;
                V_STG_ACCOUNT_DELINQ_CC_REC.CUTOFF_ID                      :=  V_CUTOFF_SEQUENCE;
                V_STG_ACCOUNT_DELINQ_CC_REC.LANGUAGE_1                     :=  V_PWC_BANK_REPORT_PARAM_REC.LANGUAGE_1;
                V_STG_ACCOUNT_DELINQ_CC_REC.LANGUAGE_2                     :=  V_PWC_BANK_REPORT_PARAM_REC.LANGUAGE_2;
                V_STG_ACCOUNT_DELINQ_CC_REC.LANGUAGE_3                     :=  V_PWC_BANK_REPORT_PARAM_REC.LANGUAGE_3;
                V_STG_ACCOUNT_DELINQ_CC_REC.CURRENCY_CODE                  :=  V_CURRENCY_TABLE_RECORD.CURRENCY_CODE;
                V_STG_ACCOUNT_DELINQ_CC_REC.CURRENCY_EXPONENT              :=  V_CURRENCY_TABLE_RECORD.CURRENCY_EXPONENT;
                V_STG_ACCOUNT_DELINQ_CC_REC.STATISTICS_PERIOD              :=  TRUNC(P_BUSINESS_DATE,'MONTH');
                V_STG_ACCOUNT_DELINQ_CC_REC.CLASS                          :=  V_CUR_CARD_TYPE_REC.CLASS;
                V_STG_ACCOUNT_DELINQ_CC_REC.CLASS_GROUP                    :=  V_CUR_CARD_TYPE_REC.CLASS_GROUP;
                V_KEY_VAL := '';
                CASE    V_STG_ACCOUNT_DELINQ_CC_REC.CLASS_GROUP
                WHEN    'IN'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS_GROUP.IN';
                WHEN    'BU'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS_GROUP.BU';
                ELSE    V_KEY_VAL := 'NA';
                END CASE;
                V_BUNDLE_REPORT_V3 := PCRD_REPORTING_TOOLS_V3.NEW_BUNDLE_REPORT_V3;
                V_BUNDLE_REPORT_V3.KEY_VAL                      := V_KEY_VAL;
                V_BUNDLE_REPORT_V3.LANGUAGE_1                   := V_STG_ACCOUNT_DELINQ_CC_REC.LANGUAGE_1;
                V_BUNDLE_REPORT_V3.LANGUAGE_2                   := V_STG_ACCOUNT_DELINQ_CC_REC.LANGUAGE_2;
                V_BUNDLE_REPORT_V3.LANGUAGE_3                   := V_STG_ACCOUNT_DELINQ_CC_REC.LANGUAGE_3;
                RETURN_STATUS   :=  PCRD_REPORTING_TOOLS_V3.GET_REPORT_LABELS   ( V_BUNDLE_REPORT_V3 );
                IF RETURN_STATUS <> DECLARATION_CST.OK
                THEN
                    V_ENV_INFO_TRACE.USER_MESSAGE   :=  'ERROR RETURNED BY PCRD_REPORTING_TOOLS_V3.GET_REPORT_LABELS'  ||
                                                        ',BUNDLE : ' || PCRD_REPORTING_TOOLS_V3.BUNDLE ||
                                                        ',KEY_VAL : ' || V_KEY_VAL;
                    PCRD_GENERAL_TOOLS.PUT_TRACES  (V_ENV_INFO_TRACE, $$PLSQL_LINE  );
                    RETURN  ( DECLARATION_CST.ERROR );
                END IF;
                V_STG_ACCOUNT_DELINQ_CC_REC.CLASS_GROUP_DESC_1             :=  V_BUNDLE_REPORT_V3.VALUE_1;
                V_STG_ACCOUNT_DELINQ_CC_REC.CLASS_GROUP_DESC_2             :=  V_BUNDLE_REPORT_V3.VALUE_2;
                V_STG_ACCOUNT_DELINQ_CC_REC.CLASS_GROUP_DESC_3             :=  V_BUNDLE_REPORT_V3.VALUE_3;
                V_KEY_VAL := '';
                CASE    V_STG_ACCOUNT_DELINQ_CC_REC.CLASS
                WHEN    '01'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS.1';
                WHEN    '02'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS.2';
                WHEN    '03'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS.3';
                WHEN    '04'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS.4';
                WHEN    '05'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS.5';
                WHEN    '06'         THEN    V_KEY_VAL    := 'STA_01_LAC_CREDIT_QTR.CLASS.6';
                ELSE    V_KEY_VAL := 'NA';
                END CASE;
                V_BUNDLE_REPORT_V3 := PCRD_REPORTING_TOOLS_V3.NEW_BUNDLE_REPORT_V3;
                V_BUNDLE_REPORT_V3.KEY_VAL                      := V_KEY_VAL;
                V_BUNDLE_REPORT_V3.LANGUAGE_1                   := V_STG_ACCOUNT_DELINQ_CC_REC.LANGUAGE_1;
                V_BUNDLE_REPORT_V3.LANGUAGE_2                   := V_STG_ACCOUNT_DELINQ_CC_REC.LANGUAGE_2;
                V_BUNDLE_REPORT_V3.LANGUAGE_3                   := V_STG_ACCOUNT_DELINQ_CC_REC.LANGUAGE_3;
                RETURN_STATUS   :=  PCRD_REPORTING_TOOLS_V3.GET_REPORT_LABELS   ( V_BUNDLE_REPORT_V3 );
                IF RETURN_STATUS <> DECLARATION_CST.OK
                THEN
                    V_ENV_INFO_TRACE.USER_MESSAGE   :=  'ERROR RETURNED BY PCRD_REPORTING_TOOLS_V3.GET_REPORT_LABELS'  ||
                                                        ',BUNDLE : ' || PCRD_REPORTING_TOOLS_V3.BUNDLE ||
                                                        ',KEY_VAL : ' || V_KEY_VAL;
                    PCRD_GENERAL_TOOLS.PUT_TRACES  (V_ENV_INFO_TRACE, $$PLSQL_LINE  );
                    RETURN  ( DECLARATION_CST.ERROR );
                END IF;
                V_STG_ACCOUNT_DELINQ_CC_REC.CLASS_DESCRIPTION_1            :=  V_BUNDLE_REPORT_V3.VALUE_1;
                V_STG_ACCOUNT_DELINQ_CC_REC.CLASS_DESCRIPTION_2            :=  V_BUNDLE_REPORT_V3.VALUE_2;
                V_STG_ACCOUNT_DELINQ_CC_REC.CLASS_DESCRIPTION_3            :=  V_BUNDLE_REPORT_V3.VALUE_3;
                V_STG_ACCOUNT_DELINQ_CC_REC.NBR_DELINQ_ACC_LESS_30_DAYS    :=  0.;
                V_STG_ACCOUNT_DELINQ_CC_REC.VAL_DELINQ_ACC_LESS_30_DAYS    :=  0.;
                V_STG_ACCOUNT_DELINQ_CC_REC.NBR_DELINQ_ACC_OVER_120_DAYS   :=  0.;
                V_STG_ACCOUNT_DELINQ_CC_REC.VAL_DELINQ_ACC_OVER_120_DAYS   :=  0.;
                V_STG_ACCOUNT_DELINQ_CC_REC.NBR_DELINQUENT_ACC_30_60_DAYS  :=  0.;
                V_STG_ACCOUNT_DELINQ_CC_REC.VAL_DELINQUENT_ACC_30_60_DAYS  :=  0.;
                V_STG_ACCOUNT_DELINQ_CC_REC.NBR_DELINQUENT_ACC_60_90_DAYS  :=  0.;
                V_STG_ACCOUNT_DELINQ_CC_REC.VAL_DELINQUENT_ACC_60_90_DAYS  :=  0.;
                V_STG_ACCOUNT_DELINQ_CC_REC.NBR_DELINQUENT_ACC_90_120_DAYS :=  0.;
                V_STG_ACCOUNT_DELINQ_CC_REC.VAL_DELINQUENT_ACC_90_120_DAYS :=  0.;
                V_STG_ACCOUNT_DELINQ_CC_REC.TOTAL_NBR_DELINQUENT_ACCOUNTS  :=  0.;
                V_STG_ACCOUNT_DELINQ_CC_REC.TOTAL_VAL_DELINQUENT_ACCOUNTS  :=  0.;
                V_STG_ACCOUNT_DELINQ_CC_REC.NBR_DELINQ_ACC_WITH_GRACE_PER  :=  0.;
                V_STG_ACCOUNT_DELINQ_CC_REC.VAL_DELINQ_ACC_WITH_GRACE_PER  :=  0.;
                V_STG_ACCOUNT_DELINQ_CC_REC.TOT_OF_CREDIT_LIMIT_LC         :=  0.;
                V_STG_ACCOUNT_DELINQ_CC_REC.TOT_OF_CREDIT_LIMIT_UC         :=  0.;
                FOR V_CUR_DWH_ACC_DAILY_SUM_REC IN CUR_DWH_ACC_DAILY_SUMMARY (V_PWC_BANK_REPORT_PARAM_REC.BANK_CODE,V_STG_ACCOUNT_DELINQ_CC_REC.CLASS,V_STG_ACCOUNT_DELINQ_CC_REC.CLASS_GROUP)
                LOOP
                    CASE V_CUR_DWH_ACC_DAILY_SUM_REC.DPD_CLASSIFICATION_CODE
                    WHEN '0' THEN V_STG_ACCOUNT_DELINQ_CC_REC.NBR_DELINQ_ACC_LESS_30_DAYS   := V_STG_ACCOUNT_DELINQ_CC_REC.NBR_DELINQ_ACC_LESS_30_DAYS + 1;
                                  V_STG_ACCOUNT_DELINQ_CC_REC.VAL_DELINQ_ACC_LESS_30_DAYS   := V_STG_ACCOUNT_DELINQ_CC_REC.VAL_DELINQ_ACC_LESS_30_DAYS + NVL(V_CUR_DWH_ACC_DAILY_SUM_REC.OUTSTANDING_AMOUNT,0);
                    WHEN '1' THEN V_STG_ACCOUNT_DELINQ_CC_REC.NBR_DELINQ_ACC_LESS_30_DAYS   := V_STG_ACCOUNT_DELINQ_CC_REC.NBR_DELINQ_ACC_LESS_30_DAYS + 1;
                                  V_STG_ACCOUNT_DELINQ_CC_REC.VAL_DELINQ_ACC_LESS_30_DAYS   := V_STG_ACCOUNT_DELINQ_CC_REC.VAL_DELINQ_ACC_LESS_30_DAYS + NVL(V_CUR_DWH_ACC_DAILY_SUM_REC.OUTSTANDING_AMOUNT,0);
                    WHEN '2' THEN V_STG_ACCOUNT_DELINQ_CC_REC.NBR_DELINQUENT_ACC_30_60_DAYS := V_STG_ACCOUNT_DELINQ_CC_REC.NBR_DELINQUENT_ACC_30_60_DAYS + 1;
                                  V_STG_ACCOUNT_DELINQ_CC_REC.VAL_DELINQUENT_ACC_30_60_DAYS := V_STG_ACCOUNT_DELINQ_CC_REC.VAL_DELINQUENT_ACC_30_60_DAYS + NVL(V_CUR_DWH_ACC_DAILY_SUM_REC.OUTSTANDING_AMOUNT,0);
                    WHEN '3' THEN V_STG_ACCOUNT_DELINQ_CC_REC.NBR_DELINQUENT_ACC_60_90_DAYS := V_STG_ACCOUNT_DELINQ_CC_REC.NBR_DELINQUENT_ACC_60_90_DAYS + 1;
                                  V_STG_ACCOUNT_DELINQ_CC_REC.VAL_DELINQUENT_ACC_60_90_DAYS := V_STG_ACCOUNT_DELINQ_CC_REC.VAL_DELINQUENT_ACC_60_90_DAYS + NVL(V_CUR_DWH_ACC_DAILY_SUM_REC.OUTSTANDING_AMOUNT,0);
                    WHEN '4' THEN V_STG_ACCOUNT_DELINQ_CC_REC.NBR_DELINQUENT_ACC_90_120_DAYS:= V_STG_ACCOUNT_DELINQ_CC_REC.NBR_DELINQUENT_ACC_90_120_DAYS + 1;
                                  V_STG_ACCOUNT_DELINQ_CC_REC.VAL_DELINQUENT_ACC_90_120_DAYS:= V_STG_ACCOUNT_DELINQ_CC_REC.VAL_DELINQUENT_ACC_90_120_DAYS + NVL(V_CUR_DWH_ACC_DAILY_SUM_REC.OUTSTANDING_AMOUNT,0);

                    ELSE
                        V_STG_ACCOUNT_DELINQ_CC_REC.NBR_DELINQ_ACC_OVER_120_DAYS := V_STG_ACCOUNT_DELINQ_CC_REC.NBR_DELINQ_ACC_OVER_120_DAYS + 1;
                        V_STG_ACCOUNT_DELINQ_CC_REC.VAL_DELINQ_ACC_OVER_120_DAYS := V_STG_ACCOUNT_DELINQ_CC_REC.VAL_DELINQ_ACC_OVER_120_DAYS +  NVL(V_CUR_DWH_ACC_DAILY_SUM_REC.OUTSTANDING_AMOUNT,0);
                    END CASE;
                    V_STG_ACCOUNT_DELINQ_CC_REC.TOTAL_NBR_DELINQUENT_ACCOUNTS       := V_STG_ACCOUNT_DELINQ_CC_REC.TOTAL_NBR_DELINQUENT_ACCOUNTS + 1;
                    V_STG_ACCOUNT_DELINQ_CC_REC.TOTAL_VAL_DELINQUENT_ACCOUNTS       := V_STG_ACCOUNT_DELINQ_CC_REC.TOTAL_VAL_DELINQUENT_ACCOUNTS + NVL(V_CUR_DWH_ACC_DAILY_SUM_REC.OUTSTANDING_AMOUNT,0);
                    V_STG_ACCOUNT_DELINQ_CC_REC.TOT_OF_CREDIT_LIMIT_LC              := V_STG_ACCOUNT_DELINQ_CC_REC.TOT_OF_CREDIT_LIMIT_LC + NVL(V_CUR_DWH_ACC_DAILY_SUM_REC.CREDIT_LIMIT_APPROVED_AMOUNT,0);
                END LOOP;
                RETURN_STATUS   :=  PCRD_STA_REPORTING_TOOLS_1_V3.PUT_ON_STG_ACCOUNT_DELINQ_CC(V_STG_ACCOUNT_DELINQ_CC_REC);
                IF (RETURN_STATUS <> DECLARATION_CST.OK)
                THEN
                    V_ENV_INFO_TRACE.USER_MESSAGE := 'ERROR RETURNED BY PCRD_STA_REPORTING_TOOLS_1_V3.PUT_ON_STG_ACCOUNT_DELINQ_CC, STATISTICS_PERIOD = ' ||  V_STG_ACCOUNT_DELINQ_CC_REC.STATISTICS_PERIOD ||
                                        ', BANK_CODE = ' || V_STG_ACCOUNT_DELINQ_CC_REC.BANK_CODE ||
                                        ', STATISTICS_PERIOD = ' || V_STG_ACCOUNT_DELINQ_CC_REC.STATISTICS_PERIOD ||
                                        ', CLASS = ' || V_STG_ACCOUNT_DELINQ_CC_REC.CLASS ||
                                        ', CLASS_GROUP = ' || V_STG_ACCOUNT_DELINQ_CC_REC.CLASS_GROUP ||
                                        ', NETWORK_CODE = ' || V_STG_ACCOUNT_DELINQ_CC_REC.NETWORK_CODE ;
                    PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
                    RETURN (DECLARATION_CST.ERROR);
                END IF;
            END LOOP;
        END LOOP;
    END IF;
    COMMIT;
    RETURN(DECLARATION_CST.OK);
EXCEPTION WHEN OTHERS
THEN
    V_ENV_INFO_TRACE.USER_MESSAGE   :=  NULL;
    PCRD_GENERAL_TOOLS.PUT_TRACES (V_ENV_INFO_TRACE,$$PLSQL_LINE);
    RETURN(DECLARATION_CST.NOK);
END PREP_STG_ACCOUNT_DELINQ_DWH_CC;
-----------------------------------------------------------------------------------------------------------------------------------------------
END PCRD_STA_REPORTING_TOOLS_1_V3;
/


-- END OF DDL SCRIPT FOR PACKAGE BODY SIAM1_VALBO.PCRD_STA_REPORTING_TOOLS_1_V3

