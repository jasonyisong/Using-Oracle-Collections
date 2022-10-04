# Using-Oracle-Collections Example

## 1. Create Type

```
CREATE TYPE MY_CARD AS OBJECT (CARD_NAME VARCHAR2(25),CARD_NUM NUMBER);

CREATE TYPE MY_CARD_NST AS TABLE OF MY_CARD;

```

## 2. Create Table

```
CREATE TABLE  "MY_WALLET" 
   (    
    "ID" NUMBER GENERATED BY DEFAULT ON NULL AS IDENTITY MINVALUE 1 MAXVALUE 9999999999999999999999999999 INCREMENT BY 1 START WITH 1 CACHE 20 NOORDER  NOCYCLE  NOKEEP  NOSCALE  NOT NULL ENABLE, 
    "CARD_TYPE" VARCHAR2(20) COLLATE "USING_NLS_COMP", 
    "NOTES" VARCHAR2(200) COLLATE "USING_NLS_COMP", 
    "CARDS"  "MY_CARD_NST" , 
     CONSTRAINT "MY_WALLET_PK" PRIMARY KEY ("ID")
  USING INDEX  ENABLE
   )  DEFAULT COLLATION "USING_NLS_COMP" 
 NESTED TABLE "CARDS" STORE AS "CARD_STORE_TAB"
 RETURN AS VALUE
/

```

## 3. Insert Demo data

```
INSERT INTO MY_WALLET (CARD_TYPE) VALUES ('Credit Card');

INSERT INTO MY_WALLET (CARD_TYPE) VALUES ('Library Card');

```

## 4. Create Package

```
CREATE OR REPLACE PACKAGE MAG_CARD_PKG
IS
    PROCEDURE ADD_CARD_INFO (P_CARD_TYPE_ID NUMBER,P_CARD_NAME VARCHAR2,P_CARD_NUM NUMBER);
    PROCEDURE DISPLAY_CARD_INFO (P_CARD_TYPE_ID NUMBER);
    
END;
```

## 5. Create Package Body

```

CREATE OR REPLACE PACKAGE BODY MAG_CARD_PKG
IS
    PROCEDURE ADD_CARD_INFO (P_CARD_TYPE_ID NUMBER,P_CARD_NAME VARCHAR2,P_CARD_NUM NUMBER)
    IS
        V_CARD_INFO MY_CARD_NST;
        I INTEGER;
    BEGIN
        SELECT CARDS INTO V_CARD_INFO FROM MY_WALLET WHERE ID = P_CARD_TYPE_ID;
        -- if card exist
        IF V_CARD_INFO.EXISTS(1) THEN
            I := V_CARD_INFO.LAST;
            -- add a new card after this
            V_CARD_INFO.EXTEND(1);
            V_CARD_INFO(I+1) := MY_CARD(P_CARD_NAME,P_CARD_NUM);
            UPDATE MY_WALLET SET CARDS=V_CARD_INFO WHERE ID=P_CARD_TYPE_ID;
        ELSE
            -- if no card exist, add this card
            UPDATE MY_WALLET SET CARDS=MY_CARD_NST(MY_CARD(P_CARD_NAME,P_CARD_NUM)) WHERE ID=P_CARD_TYPE_ID;
        END IF;
    END;
    
    PROCEDURE DISPLAY_CARD_INFO (P_CARD_TYPE_ID NUMBER)
    IS
        V_CARD_INFO MY_CARD_NST;
        I INTEGER;
    BEGIN
        SELECT CARDS INTO V_CARD_INFO FROM MY_WALLET WHERE ID=P_CARD_TYPE_ID;
        -- if card exist
        IF V_CARD_INFO.EXISTS(1) THEN
            -- loop all cards
            FOR IDX IN V_CARD_INFO.FIRST..V_CARD_INFO.LAST LOOP
                -- print this card
                HTP.P('MY CARD NAME:' || V_CARD_INFO(IDX).CARD_NAME || ', CARD NO: ' || V_CARD_INFO(IDX).CARD_NUM );
            END LOOP;
        ELSE
            HTP.P('NO SUCH TYPE OF CARD IN MY WALLET.');
        END IF;
    END;
    
END;

```

## 6. Add and Display Collections Data

```
BEGIN
    -- card type 1 (Credit Card)
    MAG_CARD_PKG.ADD_CARD_INFO(1,'AIB',123435353);
    MAG_CARD_PKG.ADD_CARD_INFO(1,'BOI',645345233);
    -- card type 2 (Library Card)
    MAG_CARD_PKG.ADD_CARD_INFO(2,'Central Library',4235234324);
    -- Show all cards with card type 1 (Credit Card)
    MAG_CARD_PKG.DISPLAY_CARD_INFO(1);
END;
```

## 7. SQL select Collections Data

```
SELECT C1.ID,C1.CARD_TYPE,C2.*  FROM MY_WALLET C1,TABLE(C1.CARDS) C2

```
