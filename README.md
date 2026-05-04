# Sistem za upravljanje skladištima maloprodajnih objekata

Danilo Markovic, RA 109/2016

## Motivacija
Kako bi maloprodajni objekti mogli da maksimizuju profite,
neophodno je da posvete pažnju da na lageru imaju proizvode koji se dobro prodaju i u prikladnim količinama.

Ukoliko bi mušterija kupila neki proizvod kog trenutno nema u prodavnici, to je izgubljen potencijalni profit koji bi nastao prodajom.

Takođe, ukoliko nekog proizvoda trenutno ima previše na lageru, a ne prodaje se u značajnim količinama,
to je prostor koji se mogao iskoristiti za neki drugi proizvod, što opet predstavlja neiskorišćen potencijalni profit.

Dakle, u svrhu optimizovanja iskorišćenja skladišnog prostora imalo bi smisla koristiti ekspertski sistem
koji bi mogao dati uvid u istoriju protoka proizvoda i pružiti pomoć pri odlučivanju kada dopuniti zalihe, kojih proizvoda i u kojim količinama.

## Opis sistema
Kako je ovo planirano da bude proof of concept jednog sistema za upravljanje skladištem,
radićemo sa nekim pretpostavkama koje će pojednostaviti domen problema i staviti u fokus samu logiku za upravljanje skladištem.

Predlog je da sistem predstavimo kao prodavnicu koja ima svoje skladište proizvoda i kupce koji žele da vrše kupovinu, tako da bi naš sistem imao dva tipa korisnika, kupca i administratora skladišta.

Kupac može nakon logovanja da pregleda inventar prodavnice, bira proizvode koje želi da doda u korpu, i izvrši kupovinu.

Administrator skladišta ima uvid u stanje proizvoda na lageru kroz vreme i mogućnost poručivanja od dobavljača na kraju dana.

Iako se sistem projektuje za maloprodajne objekte, da bismo sakupili podatke o protoku proizvoda predlog je da se kupovina obavlja takođe preko veb sajta.

Pošto je akcenat na skladištu nećemo proveravati stanje računa kupca prilikom kupovine,
nećemo stavljati granicu na količinu proizvoda koje može stati u skladište,
i pretpostavićemo da dobavljači uvek imaju proizvod na stanju i da mogu da obave dostavu u periodu od zatvaranja prodavnice do otvaranja sledećeg dana.

Dakle, planirani input sistema je količina različitih proizvoda na lageru i broj njihovih kupovina u prošlosti,
a output je predlog količine proizvoda koji dobaviti od dobavljača za sledeći dan.

Plan je da se protok vremena simulira preko administratorske stranice, gde bi administrator imao dugme koje kaže nešto poput "close shop for the day".
Pritiskom na to dugme bi adminu prikazalo stranicu na kojoj bi naš sistem dao preporuke o količinama za dopunu,
i nakon verifikacije, potencijalne izmene i potvrde, prešlo bi se na sledeći dan gde su dostvaljači već dostavili traženu količinu proizvoda.

U svrhu bržeg testiranja planirano je dodati opciju za nasumično generisanje kupovina,
gde bi se na kraju dana kreirale kupovine u kojima frekvencija kupljenih proizvoda prati neku distribuciju, na primer normalnu ili stepenu.


## Klase koje će predstavljati sistem
- Article - predstavlja jedan specifičan artikal tj. proizvod
    - id
    - name 
- Stock - predstavlja kombinaciju jednog artikla i njegove količine u skladištu
    - article
    - quantity
    - shouldReplenish = true / false
- Warehouse - predstavlja listu Stock-ova u našem skladištu
    - stockList
- OrderLine - predstavlja kombinaciju artikla i njegove količine prilikom jedne kupovine
    - article
    - quantity
- Order - predstavlja listu OrderLine-ova za jednu kupovinu
    - id
    - orderLines
    - timestamp
- ReplenishmentSuggestion - predstavlja predlog za administratora, koje proizvode treba da naruči i u kojoj količini
    - id
    - article
    - suggestedQuantity
    - isApproved


## Primer rezonovanja
- Forward chaining
    1. order event -> decrement quantity in stock by x
    
    2. rule sa watch postavljenim na quantity polje

        for each stock where quantity < productsSoldInLastWeek * 2 -> set stock shouldReplenish to true
    
    3. rule sa watch postavljenim na shouldReplenish polje

        for each stock where shouldReplenish == true -> create ReplenishmentSuggestion with suggestedQuantity = varira na osnovu potražnje koji ce se izracunati nekom formulom
        
        Administratoru izlaze svi ReplenishmentSuggestions na EOD stranici,
        kad odobri predlog, promeni se status od ReplenishmentSuggestion-a u isApproved = true
    
    4. for each suggested_replenishment
            
        where isApproved == true -> replenish that amount else delete suggested_replenishment

- Backward chaining + CEP
    - Admin može da postavi pitanje da li treba artikal izbaciti iz stock-a, gde naš sistem treba na osnovu potražnje artikala kroz duži vremenski period da proceni da li se isplati i dalje ga poručivati
