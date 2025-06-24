# Projekt_2_PowerBI_Potraviny_Miroslav_Coufalik
# Projekt Power BI: Analýza dostupnosti potravin v ČR a srovnání s EU

## Identifikace projektu

**Název Power BI souboru:** `Projekt_2_PowerBI_Potraviny_Miroslav_Coufalik.pbix`

## Úvod a cíl projektu

Tento projekt navazuje na předchozí SQL analýzu dostupnosti potravin. Cílem této části bylo vizualizovat připravené datové sady v prostředí Microsoft Power BI a vytvořit interaktivní report splňující definovaná kritéria. Důraz byl kladen na grafické zobrazení vývoje mezd, cen potravin, jejich vzájemné dostupnosti a kontextuální srovnání s makroekonomickými ukazateli ČR a dalších evropských zemí.

## Použité datové zdroje a datový model

Report v Power BI využívá dvě hlavní tabulky, které byly výsledkem předchozí SQL datové přípravy. Tyto tabulky byly do Power BI načteny (např. pomocí CSV exportu z databáze nebo přímým databázovým připojením).

1.  **`t_miroslav_coufalik_project_sql_primary_final_2`** (v Power BI modelu): Obsahuje data o průměrných ročních mzdách (celkově za ČR a po odvětvích) a průměrných ročních cenách vybraných potravin v ČR za období 2006-2018.
    *   Klíčové sloupce: `year`, `industry_name`, `avg_salary`, `food_category_name`, `avg_price_food`.
2.  **`t_miroslav_coufalik_project_sql_secondary_final_2`** (v Power BI modelu): Obsahuje roční údaje o HDP, GINI koeficientu a populaci pro evropské státy (včetně ČR) za období 2006-2018.
    *   Klíčové sloupce: `country`, `year`, `GDP`, `gini`, `population`.

V datovém modelu Power BI byl mezi těmito dvěma tabulkami vytvořen **vztah typu mnoho-ku-mnoha (M:N)** přes společný sloupec `year`.

## Struktura a obsah Power BI Reportu

Report je strukturován do tří stránek, každá zaměřená na specifickou oblast analýzy:

**Stránka 1: "Mzdy a Ceny ČR"**
*   **Obsah:** Vizualizace vývoje průměrných mezd v ČR (celkově a podle odvětví) a vývoje průměrných cen vybraných kategorií potravin v čase.
*   **Použité vizuály:** Spojnicový graf (vývoj celostátní mzdy), Seskupený sloupcový graf (mzdy v odvětvích v r. 2018), Spojnicový graf (vývoj cen potravin), Karta (celostátní průměrná mzda v r. 2018).
*   **Interaktivita:** Průřez (Slicer) pro filtrování podle roku.

**Stránka 2: "Kupní síla a Potraviny"**
*   **Obsah:** Analýza kupní síly obyvatelstva ve vztahu k vybraným základním potravinám (chléb, mléko) a srovnání průměrných cen různých kategorií potravin.
*   **Použité vizuály:** Seskupený sloupcový graf (kupní síla pro chléb a mléko v letech 2006 a 2018), Seskupený pruhový graf (srovnání průměrných cen potravin v r. 2018, seřazeno).
*   **Interaktivita:** Průřez (Slicer) pro filtrování podle kategorií potravin (ovlivňuje pruhový graf). Interakce sliceru byla upravena tak, aby neovlivňovala graf kupní síly.

**Stránka 3: "EU Srovnání a HDP"**
*   **Obsah:** Zařazení ČR do kontextu ostatních evropských zemí pomocí makroekonomických ukazatelů a analýza vztahu mezi HDP a vývojem mezd v ČR.
*   **Použité vizuály:** Mapa (zobrazení GINI koeficientu nebo HDP pro evropské země v r. 2018), Kombinovaný spojnicový a skládaný sloupcový graf (vývoj HDP ČR a průměrné mzdy ČR), Matice (zobrazení HDP a GINI pro jednotlivé země a roky s využitím hierarchie).
*   **Interaktivita:** Průřez (Slicer) pro filtrování podle země (ovlivňuje mapu a matici).

## Implementované DAX výpočty

Pro rozšíření analytických možností byly vytvořeny následující kalkulované sloupce a míry:

**Kalkulované sloupce (v tabulce `t_miroslav_coufalik_project_sql_primary_final_2`):**

1.  **`YearAsText`**: Převede číselnou hodnotu roku na textový řetězec.
    ```dax
    YearAsText = FORMAT('t_miroslav_coufalik_project_sql_primary_final_2'[year], "0")
    ```
2.  **`Kategorie Mzdy`**: Kategorizuje průměrnou mzdu (`avg_salary`) do předdefinovaných úrovní (Nízká, Střední, Vysoká) pro snazší segmentaci. Hranice byly nastaveny na <20k, 20k-35k, >=35k.
    ```dax
    Kategorie Mzdy =
    VAR CurrentSalary = 't_miroslav_coufalik_project_sql_primary_final_2'[avg_salary]
    RETURN
        IF(
            ISBLANK(CurrentSalary), 
            "Neznámá (chybí údaj)", 
            SWITCH(
                TRUE(),
                CurrentSalary < 20000, "Nízká mzda (<20k)",
                CurrentSalary < 35000, "Střední mzda (20k-35k)",
                CurrentSalary >= 35000, "Vysoká mzda (>=35k)",
                "Neznámá (jiná chyba)" 
            )
        )
    ```

**Míry (Measures):**

1.  **`Počet kategorií potravin`**: Spočítá počet unikátních kategorií potravin v datasetu.
    ```dax
    Počet kategorií potravin = DISTINCTCOUNT('t_miroslav_coufalik_project_sql_primary_final_2'[food_category_name])
    ```
2.  **`Kupní síla`**: Vypočítá, kolik jednotek dané potraviny (v aktuálním kontextu filtru) lze koupit za celostátní průměrnou mzdu. Zajišťuje, že pro výpočet se vždy použije celostátní průměrná mzda, nezávisle na filtru kategorie potravin.
    ```dax
    Kupní síla = 
    VAR CelostatniMzda = CALCULATE(
                            AVERAGE('t_miroslav_coufalik_project_sql_primary_final_2'[avg_salary]),
                            't_miroslav_coufalik_project_sql_primary_final_2'[industry_name] = "Czech Republic Overall",
                            ALL('t_miroslav_coufalik_project_sql_primary_final_2'[food_category_name])
                        )
    VAR PrumernaCenaPotraviny = AVERAGE('t_miroslav_coufalik_project_sql_primary_final_2'[avg_price_food])
    RETURN
        DIVIDE(CelostatniMzda, PrumernaCenaPotraviny)
    ```

## Splnění zadaných kritérií

*   **Rozsah 2-3 stránky:** Report obsahuje 3 stránky.
*   **Použití minimálně 5 různých typů vizuálů:** Bylo použito 8 různých typů vizuálů (spojnicový, seskupený sloupcový, seskupený pruhový, karta, průřez, mapa, kombinovaný graf, matice).
*   **Filtrování pomocí slicerů:** Na všech stránkách jsou implementovány relevantní slicery.
*   **Využití bookmarks/page navigation:** Pro navigaci mezi stránkami byl použit vizuál "Navigátor stránek".
*   **Propojení více datových zdrojů:** Dvě hlavní tabulky (`primary_final_2` a `secondary_final_2`) jsou propojeny v datovém modelu.
*   **Použití datové hierarchie o alespoň dvou úrovních:** V matici na 3. stránce je implementována hierarchie `country` -> `year`.
*   **Vytvoření alespoň 1 measure a 1 kalkulovaného sloupce:** Byly vytvořeny 2 kalkulované sloupce a 2 míry.
*   **Grafická úprava použitých vizuálů a vizuálně přívětivý výsledný report:** Byla provedena základní grafická úprava (jednotný motiv, názvy vizuálů, čitelnost os) pro zajištění přehlednosti a funkčnosti reportu.

## Závěr

Výsledný Power BI report poskytuje interaktivní pohled na analyzovaná data a splňuje všechna zadaná kritéria. Umožňuje exploraci vývoje mezd, cen, kupní síly a základní mezinárodní srovnání.

---

## Aktualizace a vylepšení na základě zpětné vazby 24.06.2025

Na základě připomínek z hodnocení lektora byly v reportu Power BI provedeny následující úpravy a vylepšení:

**1. Datový model - Kalendářová tabulka:**
*   Byla vytvořena nová dimenzionální tabulka `DimRok` pomocí DAX, obsahující unikátní roky z datových zdrojů.
*   Původní přímý M:N vztah mezi hlavními datovými tabulkami (`primary_final_2` a `secondary_final_2`) přes sloupec `year` byl odstraněn.
*   Obě hlavní datové tabulky jsou nyní propojeny s `DimRok` vztahem 1:N.
*   Všechny vizuály a slicery využívající rok nyní odkazují na sloupec `DimRok[Rok]`, což zajišťuje konzistentní časovou dimenzi a je v souladu s best practices.

**2. Formátování a vzhled:**
*   **Fonty:** Font DIN v nadpisech byl nahrazen standardním fontem (např. Segoe UI - výchozí Power BI), který korektně zobrazuje české znaky.
*   **Formátování měny:** Sloupce a míry zobrazující finanční hodnoty (mzdy, ceny, HDP) byly naformátovány jako měna (Kč) s příslušným počtem desetinných míst. Toto formátování je nyní viditelné na osách vizuálů i v kartách.
*   **Zarovnání a rozložení:** Byla provedena úprava zarovnání a rozmístění vizuálů na jednotlivých stránkách pro lepší vizuální přehlednost a minimalizaci nevyužitého prostoru.

**3. Logika vizuálů, metrik a nadpisy:**
*   **Agregace dat:** Ve většině vizuálů byla agregace pro číselné hodnoty (mzdy, ceny, GINI, HDP) změněna ze "Součet" na **"Průměr"**, aby se zobrazovaly relevantnější a smysluplnější ukazatele, jak doporučil lektor.
*   **Nadpisy vizuálů:** Všechny nadpisy vizuálů byly revidovány a upraveny tak, aby byly více vypovídající, konkrétní a jasně popisovaly zobrazená data (např. specifikace "pouze pro ČR" u grafu HDP a mezd).
*   **Vizuál "Kupní síla":** Míra `Kupní síla` byla zkontrolována a graf upraven tak, aby správně zobrazoval rozdílné hodnoty kupní síly pro chléb a mléko v letech 2006 a 2018. Interakce se slicerem "Jídlo" byla nastavena tak, aby tento graf neovlivňoval a ten zůstal fixován na dané potraviny a roky.
*   **Matice (EU Srovnání):** Agregace pro GINI koeficient byla změněna na "Průměr". Pro HDP je použit "Průměr" (nebo "Součet" s upřesňujícím nadpisem matice).
*   **Mapa (EU Srovnání):** Velikost bublin je nyní určena **průměrnou hodnotou GINI koeficientu** (pomocí vytvořené míry `Průměr GINI`), nikoliv počtem záznamů.

**4. Interaktivita:**
*   **Chybějící slicery (Stránka 1):**
    *   Byl přidán slicer "Odvětví" (`industry_name`), který ovlivňuje graf srovnání mezd v odvětvích.
    *   Byl přidán slicer "Typ potraviny" (`food_category_name`), který ovlivňuje graf vývoje cen potravin.
    *   Interakce těchto nových slicerů byly nastaveny tak, aby neovlivňovaly ostatní vizuály na stránce, které mají zobrazovat celostátní nebo fixní data (např. karta s celostátní průměrnou mzdou, graf vývoje celostátní mzdy).
*   **Karta "Průměrný roční výdělek":** Byla upravena tak, aby zobrazovala celostátní průměrnou mzdu a nereagovala na slicery "Odvětví" a "Typ potraviny", ale pouze na slicer "Filtr roků".

**Závěr aktualizace:**
Všechny hlavní připomínky z hodnocení byly adresovány s cílem vylepšit datový model, relevanci zobrazených informací, uživatelskou interaktivitu a celkovou vizuální stránku reportu.

---

SQL skripty použité pro přípravu zdrojových dat pro tento Power BI report jsou dostupné v:
*   `/01_create_database_objects.sql`
*   `/02_research_questions.sql`
*   https://github.com/88barcode88/Dbeaver_Kurz_Projekty
