Generel information
-------------------
Java-bibliotek til parsning mv. af XBRL.
Java 7 er brugt til compilering.
Biblioteket har følgende eksterne afhængigheder:
  * joda-time:joda-time:2.9.8
  * com.fasterxml.jackson.core:jackson-annotations:2.9.1

Hvis JSon-funktionalitet bruges skal følgende afhængigheder også være tilstede:
  * com.fasterxml.jackson.core:jackson-core:2.9.1
  * com.fasterxml.jackson.core:jackson-databind:2.9.1
  * com.fasterxml.jackson.datatype:jackson-datatype-joda:2.9.1

Denne readme-fil er UTF-8 encodet.


Kendte fejl
-----------
a) RIV-1021
Aktuelle fakta i ErstMetadata kan pt. også indeholde tal fra ældre perioder. Dette skyldes at fakta med nedenstående dimensioner ikke er sorteret fra.
  * cmn:TwoYearsAgoMember
  * cmn:ThreeYearsAgoMember
  * cmn:FourYearsAgoMember


Introduktion
------------
Dette er et bibliotek til at parse og udtrække facts fra et XBRL-regnskab med en dansk taksonomi.
Biblioteket består af tre hoveddele jf. også vedlagte diagram (udtraekbibliotek-struktur.png).

a) Parsning af XBRL XML
Input er her et XBRL regnskab og output er en objektrepræsentation af XML-strukturen. Bemærk at segmenter ikke medtages.

b) Marshal/unmarshal af XBRL repræsenteret som JSon
Funktionalitet til at marshalle og unmarshalle XBRL til JSon (klassen BaseReport). JSon er inspireret af xbrl-json standarden, men er ikke compliant.
Denne bruges internt i Erhvervsstyrelsen men er ikke umiddelbart tænkt til eksternt brug.

b) Generering af ERST XBRL
ERST XBRL er betegnelsen for en intern model af XBRL (ErstXbrlInstance-klassen), som retter sig mod udtræk af data. Den kan genereres på to måder:

  * Via XBRL idet XBRL-dokumentet først parses
  * Via BaseReport idet JSon først unmarshalles

ERST XBRL består af:
  1) Liste af fakta (klassen XbrlFact) som er denormaliserede XBRL-data (mere præcist: Item's fra XBRL)
  2) Taxonomidata (klassen Entrypoint) som beskriver hvilken taxonomi som det pågældende XBRL-dokument anvender
  3) Metadata (klassen ErstMetadata). Består af enkeltdata (som feks. cvrnummer mv) og lister med XbrlFact's for henholdssvis aktuelle periode og forrige periode opdelt på Solo og Consolidated. Dette gør fremsøgning af relevante facts nemmere
  4) Externe metadata, som ikke kan findes ud fra XBRL (disse data findes kun internt i Erhvervsstyrelsen)


Udtræk af XBRL data
-------------------
Som input til et udtræk angives en liste af udtræksparametre (klassen UdtraekParameter). Se Javadoc for denne klasse for en nærmere beskrivelse.

Output fra udtrækket er en liste af XbrlFact's. Følgende bemærkninger kan gøres:
  * For et bestemt navn på et faktum kan der returneres flere XbrlFact's. Det kan være pga. dubletter i XBRL eller at man har ønsket at få alle tal ud uanset præcision.
  * Hvis et regnskab har både Solo og Consolidated i sig, da er det som udgangspunkt Consolidated-tal der returneres. Dette kan ændres ved eksplicit at sætte soloIConsolidated til true
  * Hvis et faktum er angivet uden dimension, da returneres kun de fakta som IKKE har en dimension (udover Consolidated eller Solo dimension)
  * For IFRS-regnskaber trækkes fakta udelukkende ud vha. "localpart" i det kvalificerede navn (men det fulde kvalificerede navn skal stadig angives). Det er for at undgå at man skal kende den konkrete version af IFRS-taxonomien (som ikke altid er tilstede)

Hvis man blot ønsker alle aktuelle fakta, så findes de som sagt i ErstMetadata.


Kode eksempler
--------------
a) Parsning af XBRL XML
import xbrludtraek.xbrl.XbrlParser
import xbrludtraek.xbrl.model.XbrlInstance
import xbrludtraek.xbrl.erstmodel.ErstXbrlInstance
import xbrludtraek.xbrl.erstmodel.ErstXbrlInstanceFactory
 
public ErstXbrlInstance parseXbrl(String xbrl) {
    XbrlParser parser = new XbrlParser(xbrl);
    XbrlInstance instans = parser.parse();
    ErstXbrlInstance erstInstans = ErstXbrlInstanceFactory.lavInstans(instans);
    return erstInstans;
}


b) Udtræk af fact uden dimension
import javax.xml.namespace.QName
import xbrludtraek.xbrl.erstmodel.ErstXbrlNames
import xbrludtraek.xbrl.erstmodel.ErstXbrlInstance
import xbrludtraek.xbrl.erstmodel.udtraek.UdtraekParameter
import xbrludtraek.xbrl.erstmodel.udtraek.UdtraekXbrlFacts
 
public List<XbrlFact> udtraek(ErstXbrlInstance erstInstans) {
    UdtraekXbrlFacts udtraek = new UdtraekXbrlFacts(erstInstans.metadata);
    UdtraekParameter param = new UdtraekParameter();
    param.navn = new QName(ErstXbrlNames.FSA_NS, "Provisions");
    List<XbrlFact> udtraekListe = udtraek.udtraek([param]);
    return udtraekListe;
}


c) Udtræk af sammenligningstal med dimension
import javax.xml.namespace.QName
import xbrludtraek.xbrl.erstmodel.ErstXbrlNames
import xbrludtraek.xbrl.erstmodel.ErstXbrlInstance
import xbrludtraek.xbrl.erstmodel.udtraek.UdtraekParameter
import xbrludtraek.xbrl.erstmodel.udtraek.UdtraekXbrlFacts
 
public List<XbrlFact> udtraek(ErstXbrlInstance erstInstans) {
    UdtraekXbrlFacts udtraek = new UdtraekXbrlFacts(erstInstans.metadata);
    UdtraekParameter param = new UdtraekParameter();
    param.navn = new QName(ErstXbrlNames.FSA_NS, "Provisions");
    param.sammenligningstal = true;
    param.dimension = new QName(ErstXbrlNames.FSA_NS, "ResultDistributionDimension");
    param.explicitDimensionValue = new QName(ErstXbrlNames.FSA_NS, "RetainedEarningsMember");
    List<XbrlFact> udtraekListe = udtraek.udtraek([param]);
    return udtraekListe;
}
