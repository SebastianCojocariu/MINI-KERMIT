Sebastian Cojocariu 321CB UBP ACS
					                                              MINI-KERMIT


Functii ajutatoare lib.c

1)void reactualizare_check(msg *msg) : calculeaza check-ul pentru mesajul dat ca parametru si il scrie la pozitia corecta din el.

2)void set_seq(msg *t,char seq) : seteaza nr de secventa dat ca parametru in mesaj,reactualizand si casuta pentru CHECK.

3)void set_type(msg *t,char type) : seteaza nr de secventa dat ca parametru in mesaj,reactualizand si casuta pentru CHECK.

4)msg* prepare_msg(char type,char *data,int len,char seq) : functie care returneaza un mesaj specific MINI-KERMIT,de tipul type,
					cu payloadul data(len = lungimea lui data) si nr de secventa = seq,avand calculat CHECK-ul

5)int check(msg *msg) : functie care returneaza -1 daca mesajul are un CHECK scris in el gresit (raportat la continutul asupra 
caruia s-a efectuat) sau mesajul e NULL,respectiv 1 altfel.


	Idee centrala:
Folosim in ambele main-uri(ksender.c,respectiv kreceiver.c) cate un while:
	
	In sender tinem un contor pentru timeouturi consecutive,pe care il actualizam la inceput.
In interiorul acestui while,trimitem pachete pana cand primim:
	-NACK : caz in care ,in functie de current_seq actualizam sau nu acest current_seq si trimitem pachetul corespunzator 
		(cel anterior sau cel cu noul current_seq actualizat);
		Mai mult,reactualizam nr la 0.(pentru a nu avea o situatie de genul TIMEOUT NACK NACK si sa ies din program)
	
	-3 TIMEOUTURI consecutive : caz in care inchid conexiunea.
Daca am iesit din acest while inseamna ca am primit ACK : caz in care,in functie de numarul de seq primit in recv si cel local 
(current_seq) pregatesc urmatorul mesaj/raman cu cel curent (in functie de variabila i.Daca s-a ajuns la i == index ma opresc deoarece
inseamna caam trimis toate mesajele) si sar la labelul de la inceputul whileului(unde se reactualizeaza nr=0 ce contorizeaza nr 
timeouturilor consecutive) in cazul in care nu am ajuns cu nr = 3,altfel inchid conexiunea. 





	In receiver tinem un while(1).La inceput verificam daca am primit sau nu send-init(intrucat vom utiliza 3*DEFAULT_TIMEOUT 
pentru a primi SEND-INIT,respectiv DEFAULT_TIMEOUT pentru celelalte pachete).Cat timp recv = NULL(timeout) sau recv e incorect sau 
nu am primit pachetul urmator fata de cel current_seq primim pachete,altfel:
		
	if(recv == NULL) incrementam nr,altfel modificam/nu modificam current_seq(in functie de nr de secventa din recv-ul primit) si 
	initializam nr=0(pentru a nu avea o situatie de genul TIMEOUT NACK NACK urmata de intreruperea conexiunii).
	Daca s-a ajuns la nr=3 intrerupem conexiunea.
		
Daca s-a trecut de acest while,inseamna ca pachetul a fost primit corect.In functie de tipul pachetului,facem operatia aferenta:
deschidem un fisier cu numele : "recv_"+nume,scriem in aceste fisiere,etc.Dupa acest lucru construim ACK pe care il vom trimite
ulterior senderului(avem cazul in care ACK e pentru SEND-INIT si ACK e pentru restul celorlalte tipuri de pachete).Cu un while
incercam sa trimitem ACK catre sender,atata timp cat am ajuns deja la 3 timeouturi in recv din receiver,sau am primit pachetul cu nr
de secventa urmator celui pe care l-am trimis in ACK (caz in care am inteles ca senderul a primit ACK-ul).Daca primim pachetul de 
tip 'B' intrerupem conexiunea,altfel ne intoarcem la inceputul whileului,unde reactualizam nr=0. 
