# sql-customer-analysis
SQL script to create and populate a PostgreSQL database based on the Brazilian E-Commerce Public Dataset (Olist).

-- Case 1 : Sipariş Analizi
-- Question 1 : 
-- -Aylık olarak order dağılımını inceleyiniz. Tarih verisi için order_approved_at kullanılmalıdır.

SELECT
	DATE(DATE_TRUNC('month', ORDER_APPROVED_AT)) AS ORDER_DATE,
	COUNT(DISTINCT ORDER_ID) AS ORDER_COUNT
FROM
	ORDERS
GROUP BY
	1
ORDER BY
	2 desc
;

-- Siparişlerin onaylanma tarihine göre yapılan aylık analizde, 
-- e-ticaret platformunun 2017 yılının başlarından itibaren ciddi bir büyüme sürecine girdiği görülmektedir. 
-- Özellikle Kasım 2017 ve Mart 2018 aylarında sipariş sayılarında belirgin artışlar yaşanmıştır. 
-- Kasım ayında yaşanan sıçrama, Black Friday gibi kampanya dönemlerine bağlanabilirken, 
-- Mart ayındaki artış ise sistemin tam oturması veya başka bir kampanya etkisi olabilir. 
-- 2018 yılı genelinde sipariş onay sayılarının 6000’in üzerinde seyretmesi, 
-- platformun güçlü bir kullanıcı kitlesi oluşturduğuna işaret etmektedir. Eylül 2018 verisi ise eksik ya da hatalı veri içeriyor olabilir, 
-- çünkü önceki aylardaki yüksek tempoya kıyasla yalnızca 1 sipariş gözükmektedir.




-- Question 2 : 
-- -Aylık olarak order status kırılımında order sayılarını inceleyiniz. 
-- Sorgu sonucunda çıkan outputu excel ile görselleştiriniz. 
-- Dramatik bir düşüşün ya da yükselişin olduğu aylar var mı? Veriyi inceleyerek yorumlayınız.
-- farkli 2 tablo olusturulabilir okunurluk acisindan 
SELECT
	DATE (DATE_TRUNC('month', ORDER_APPROVED_AT)) AS ORDER_DATE,
	ORDER_STATUS,
	COUNT(DISTINCT ORDER_ID) AS ORDER_COUNT
FROM
	ORDERS
	where order_status= 'approved'
GROUP BY
	1,
	2
;

select * from orders
	where order_status= 'approved'

-- Question 3 : 
-- -Ürün kategorisi kırılımında sipariş sayılarını inceleyiniz. 
-- Özel günlerde öne çıkan kategoriler nelerdir? Örneğin yılbaşı, sevgililer günü…

WITH
	SPECIAL_DATES AS (
		SELECT
			O.ORDER_ID,
			P.PRODUCT_CATEGORY_NAME,
			TO_CHAR(O.ORDER_APPROVED_AT, 'YYYY-MM-DD') AS SPECIAL_DATE,
			CASE
				WHEN TO_CHAR(O.ORDER_APPROVED_AT, 'MM-DD') = '06-12' THEN 'Valentine''s Day' -- Brezilya Sevgililer Günü
				WHEN TO_CHAR(O.ORDER_APPROVED_AT, 'MM-DD') = '09-07' THEN 'Independence Day' -- Brezilya Bağımsızlık Günü
				WHEN TO_CHAR(O.ORDER_APPROVED_AT, 'MM-DD') = '10-12' THEN 'Children''s Day' -- Brezilya Çocuklar Günü
				WHEN TO_CHAR(O.ORDER_APPROVED_AT, 'MM-DD') = '12-01' THEN 'Black Friday'
				WHEN TO_CHAR(O.ORDER_APPROVED_AT, 'MM-DD') = '12-02' THEN 'Cyber Monday'
				WHEN TO_CHAR(O.ORDER_APPROVED_AT, 'MM-DD') = '12-24' THEN 'Christmas Eve' -- Global Noel Arifesi
				WHEN TO_CHAR(O.ORDER_APPROVED_AT, 'MM-DD') = '12-25' THEN 'Christmas Day' -- Global Noel Günü
				WHEN TO_CHAR(O.ORDER_APPROVED_AT, 'MM-DD') = '12-31' THEN 'New Year''s Eve' -- Global Yılbaşı Arifesi (beyaz kiyafet gelenegi)
				ELSE NULL
			END AS SPECIAL_DAY
		FROM
			ORDERS O
			JOIN ORDER_ITEMS OI ON O.ORDER_ID = OI.ORDER_ID
			JOIN PRODUCTS P ON OI.PRODUCT_ID = P.PRODUCT_ID
		WHERE
			O.ORDER_APPROVED_AT IS NOT NULL
			AND O.ORDER_STATUS NOT IN ('canceled')
			
	)
	
SELECT
	SPECIAL_DAY || ' (' || SPECIAL_DATE || ')' AS SPECIAL_DAY_WITH_DATE,									
	PRODUCT_CATEGORY_NAME,
	COUNT(DISTINCT ORDER_ID) AS ORDER_COUNT
FROM
	SPECIAL_DATES
WHERE
	SPECIAL_DAY IS NOT NULL
GROUP BY
	1,2
ORDER BY
	1,
	3 DESC
	;










-- Question 4 : 
-- -Haftanın günleri(pazartesi, perşembe, ….) ve ay günleri (ayın 1’i,2’si gibi) bazında order sayılarını inceleyiniz. 
-- Yazdığınız sorgunun outputu ile excel’de bir görsel oluşturup yorumlayınız.
--to_char (order_approved_at,'Day')

--gun kiriliminda
SELECT
	TO_CHAR(ORDER_APPROVED_AT, 'Day') AS DAY_OF_WEEK,
	COUNT(DISTINCT ORDER_ID) AS COUNT_ORDER
FROM
	ORDERS
WHERE
	ORDER_APPROVED_AT IS NOT NULL
GROUP BY
	1
;

--ay kiriliminda 
SELECT
	TO_CHAR(ORDER_APPROVED_AT, 'Month') AS DAY_OF_WEEK,
	COUNT(DISTINCT ORDER_ID) AS COUNT_ORDER
FROM
	ORDERS
WHERE
	ORDER_APPROVED_AT IS NOT NULL
GROUP BY 1
;



-- Case 2 : Müşteri Analizi 
-- Question 1 : 
-- -Hangi şehirlerdeki müşteriler daha çok alışveriş yapıyor? Müşterinin şehrini en çok sipariş verdiği şehir olarak belirleyip analizi ona göre yapınız. 

-- Örneğin; Sibel Çanakkale’den 3, Muğla’dan 8 ve İstanbul’dan 10 sipariş olmak üzere 3 farklı şehirden sipariş veriyor. 
-- Sibel’in şehrini en çok sipariş verdiği şehir olan İstanbul olarak seçmelisiniz ve Sibel’in yaptığı siparişleri İstanbul’dan 21 sipariş vermiş şekilde görünmelidir.

-- Not: Mevcut veride her müşteri yalnızca tek bir şehirden ve tek bir sipariş gerçekleştirdiği için, 
-- müşterilerin dominant şehir analizi yapılırken her müşteri kendi şehriyle eşleşmiş ve sipariş sayısı 1 olarak kalmıştır. 
-- Daha çeşitli şehir ve sipariş adetleri olması halinde daha detaylı bir analiz gerçekleştirilebilir.


WITH
	CUSTOMER_ORDER_COUNT AS (
		SELECT
			O.CUSTOMER_ID,
			C.CUSTOMER_CITY,
			COUNT(O.ORDER_ID) AS COUNT_ORDER
		FROM
			ORDERS O
			JOIN CUSTOMERS C ON O.CUSTOMER_ID = C.CUSTOMER_ID
		WHERE
			O.ORDER_STATUS NOT IN ('canceled')
		GROUP BY
			1,
			2
		ORDER BY
			2 DESC
	),
	CUSTOMER_CITY_ROW_N AS (
		SELECT
			CUSTOMER_ID,
			CUSTOMER_CITY,
			ROW_NUMBER() OVER (
				PARTITION BY
					CUSTOMER_ID
				ORDER BY
					COUNT_ORDER DESC
			) ROW_N
		FROM
			CUSTOMER_ORDER_COUNT
	)
SELECT
	CC.CUSTOMER_ID,
	CC.CUSTOMER_CITY,
	COC.COUNT_ORDER,
	CC.ROW_N
FROM
	CUSTOMER_ORDER_COUNT COC
	JOIN CUSTOMER_CITY_ROW_N CC ON COC.CUSTOMER_ID = CC.CUSTOMER_ID
	AND COC.CUSTOMER_CITY = CC.CUSTOMER_CITY
WHERE
	ROW_N = 1






-- Case 3: Satıcı Analizi
-- Question 1 : 
-- -Siparişleri en hızlı şekilde müşterilere ulaştıran satıcılar kimlerdir? Top 5 getiriniz.
-- Bu satıcıların order sayıları ile ürünlerindeki yorumlar ve puanlamaları inceleyiniz ve yorumlayınız.

WITH
	AVG_DELIVERY_TIME_PER_SELLER AS (
		SELECT
			OI.SELLER_ID,
			AVG(
				ORDER_DELIVERED_CUSTOMER_DATE - ORDER_PURCHASE_TIMESTAMP
			) AS AVG_DELIVERY_TIME,
			COUNT(DISTINCT O.ORDER_ID) AS ORDER_COUNT
		FROM
			ORDERS O
			JOIN ORDER_ITEMS OI ON O.ORDER_ID = OI.ORDER_ID
		WHERE
			O.ORDER_STATUS = 'delivered'
		GROUP BY
			1
	)
SELECT
	ADS.SELLER_ID,
	R.REVIEW_COMMENT_MESSAGE,
	R.REVIEW_SCORE,
	ADS.AVG_DELIVERY_TIME,
	ADS.ORDER_COUNT
FROM
	AVG_DELIVERY_TIME_PER_SELLER ADS
	JOIN ORDER_ITEMS OI ON ADS.SELLER_ID = OI.SELLER_ID
	JOIN ORDERS O ON OI.ORDER_ID = O.ORDER_ID
	JOIN REVIEWS R ON O.ORDER_ID = R.ORDER_ID
ORDER BY
	4 ASC
LIMIT
	5;






-- Bu veriye göre en hızlı teslimat yapan satıcılar siparişlerini yaklaşık 1–2 gün arasında teslim etmiş. 
-- Bu satıcılara ait yorumların çoğunda yorum bile yok, bu da genellikle müşterinin bir sorun yaşamadığını ve memnun kaldığını gösteriyor. 
-- Puanların yüksekliği (çoğunlukla 5) hızlı teslimatla memnuniyet arasında doğrudan bir ilişki olduğunu destekliyor.




-- Question 2 : 
-- Hangi satıcılar daha fazla kategoriye ait ürün satışı yapmaktadır? 
--  Fazla kategoriye sahip satıcıların order sayıları da fazla mı? 


SELECT
	S.SELLER_ID,
	COUNT(DISTINCT P.PRODUCT_CATEGORY_NAME) AS COUNT_CATEGORY_NAME,
	COUNT(DISTINCT OI.ORDER_ID) AS ORDER_COUNT
FROM
	SELLERS S
	JOIN ORDER_ITEMS OI ON S.SELLER_ID = OI.SELLER_ID
	JOIN PRODUCTS P ON OI.PRODUCT_ID = P.PRODUCT_ID
GROUP BY
	1
ORDER BY
	2 DESC,
	3 DESC


-- Case 4 : Payment Analizi
-- Question 1 : 
-- -Ödeme yaparken taksit sayısı fazla olan kullanıcılar en çok hangi bölgede yaşamaktadır? Bu çıktıyı yorumlayınız.

WITH
	CUSTOMER_CITY_COUNT AS (
		SELECT
			C.CUSTOMER_CITY,
			COUNT(DISTINCT C.CUSTOMER_ID) AS CUSTOMER_COUNT
		FROM
			CUSTOMERS C
		GROUP BY
			1
		ORDER BY
			2 DESC
	),
	CITY_INSTALLMENTS AS (
		SELECT
			C.CUSTOMER_CITY,
			ROUND(AVG(P.PAYMENT_INSTALLMENTS), 2) AS AVG_INSTALLMENTS,
			COUNT(DISTINCT C.CUSTOMER_ID) AS CITY_USER_COUNT
		FROM
			PAYMENTS P
			JOIN ORDERS O ON P.ORDER_ID = O.ORDER_ID
			JOIN CUSTOMERS C ON O.CUSTOMER_ID = C.CUSTOMER_ID
		GROUP BY
			1
	)
SELECT
	CI.CUSTOMER_CITY,
	CI.AVG_INSTALLMENTS,
	CC.CUSTOMER_COUNT
FROM
	CITY_INSTALLMENTS CI
	JOIN CUSTOMER_CITY_COUNT CC ON CI.CUSTOMER_CITY = CC.CUSTOMER_CITY
ORDER BY	CI.AVG_INSTALLMENTS DESC


-- Küçük şehirlerde yaşayan kullanıcılar, genellikle ödeme kolaylığı sağlamak amacıyla daha yüksek taksit seçeneklerini tercih etmektedir. 
-- Ancak bu sonuçların çoğu 1-3 kişiyle sınırlı olduğundan, 
-- daha sağlıklı bir analiz için şehirlerin müşteri sayısı göz önüne alınarak bir eşik belirlenmesi 
-- (örneğin, sadece 10+ müşterisi olan şehirler) daha dengeli bir tablo sunabilir.








-- Question 2 : 
-- -Ödeme tipine göre başarılı order sayısı ve toplam başarılı ödeme tutarını hesaplayınız. 
--En çok kullanılan ödeme tipinden en az olana göre sıralayınız.

SELECT
	PAYMENT_TYPE,
	COUNT(DISTINCT P.ORDER_ID) AS ORDER_COUNT,
	ROUND(SUM(PAYMENT_VALUE)::NUMERIC, 2) AS SUM_PAYMENT_VALUE
FROM
	PAYMENTS P
	LEFT JOIN ORDERS O ON P.ORDER_ID = O.ORDER_ID
WHERE
	ORDER_STATUS IN ('shipped', 'delivered', 'invoiced', 'approved')
GROUP BY
	1
ORDER BY
	2 DESC



-- Question 3 : 
-- -Tek çekimde ve taksitle ödenen siparişlerin kategori bazlı analizini yapınız. 
-- En çok hangi kategorilerde taksitle ödeme kullanılmaktadır?


SELECT
	P.PRODUCT_CATEGORY_NAME,
	SUM(
		CASE
			WHEN PAY.PAYMENT_INSTALLMENTS = 1 THEN 1
			ELSE 0
		END
	) AS TEK_CEKIM_SAYISI,
	SUM(
		CASE
			WHEN PAY.PAYMENT_INSTALLMENTS > 1 THEN 1
			ELSE 0
		END
	) AS TAKSITLI_SAYISI,
	COUNT(*) AS TOPLAM_ODEME
FROM
	PAYMENTS PAY
	JOIN ORDER_ITEMS OI ON PAY.ORDER_ID = OI.ORDER_ID
	JOIN PRODUCTS P ON OI.PRODUCT_ID = P.PRODUCT_ID
GROUP BY
	1
	ORDER BY
	3 DESC;

-- --Taksitli ödemeler, genellikle yüksek fiyatlı ürünlerin bulunduğu kategorilerde (elektronik, ev eşyası vb.) daha fazla tercih edilmiştir.
-- Tek çekim ödemeler ise daha uygun fiyatlı ve günlük tüketim ürünlerinin olduğu kategorilerde öne çıkmaktadır.
-- Bu durum, ödeme alışkanlıklarının ürün türüne göre değiştiğini ve kategoriye özel ödeme kampanyalarının etkili olabileceğini göstermektedir.


