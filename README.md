<div align="center">
  <h1 align="center">TP3 - Traitement d’un signal ECG</h1>
</div>


<summary>Table of Contents</summary>
  <ol>              
      <li><a href="#Objectifs">Objectifs</a></li>
      <li><a href="#Suppression-du-bruit-provoqué-par-les-mouvements-du-corps">Suppression du bruit provoqué par les mouvements du corps</a></li> 
      <li><a href="#Suppression-des-interférences-des-lignes-électriques-50Hz">Suppression des interférences des lignes électriques 50Hz</a></li> 
      <li><a href="#Amélioration-du-rapport-signal-sur-bruit">Amélioration du rapport signal sur bruit</a></li> 
      <li><a href="#Identification-de-la-fréquence-cardiaque-avec-la-fonction-d’autocorrélation">Identification de la fréquence cardiaque avec la fonction d’autocorrélation</a></li> 
  </ol>
  
  # Objectifs

> • Suppression du bruit autour du signal produit par un électrocardiographe.   
> • Recherche de la fréquence cardiaque.
>
> **Commentaires** : Il est à remarquer que ce TP traite en principe des signaux continus.
> Or, l'utilisation de Matlab suppose l'échantillonnage du signal. Il faudra donc être
> vigilant par rapport aux différences de traitement entre le temps continu et le temps
> discret.
> 
> **Tracé des figures** : toutes les figures devront être tracées avec les axes et les
> légendes des axes appropriés.
> 
> **Travail demandé** : un script Matlab commenté contenant le travail réalisé et des
> commentaires sur ce que vous avez compris et pas compris, ou sur ce qui vous a
> semblé intéressant ou pas, bref tout commentaire pertinent sur le TP.
  

# Suppression-du-bruit-provoqué-par-les-mouvements-du-corps

1- Sauvegarder le signal ECG sur votre répertoire de travail, puis charger-le dans Matlab à l’aide la commande **load**.

```matlab
load("ecg.mat");
x = ecg;
```
2- Ce signal a été échantillonné avec une fréquence de 500Hz. Tracer-le en fonction du temps, puis faire un zoom sur une période du signal.

```matlab
fs = 500;
N = length(x)
ts = 1/fs
%tracer ECG en fonction de temps
t = (0:N-1)*ts;
plot(t,x);
title("le signal ECG");
```
![ecg](https://user-images.githubusercontent.com/121026639/210149705-09a71f0f-08c8-43c1-93c9-60d0acef9aed.png)


3- Pour supprimer les bruits à très basse fréquence dues aux mouvements du corps, on utilisera un filtre idéal passe-haut. Pour ce faire, calculer tout d’abord la TFD du
signal ECG, régler les fréquences inférieures à 0.5Hz à zéro, puis effectuer une TFDI pour restituer le signal filtré. 

```matlab
%TFD
y = fft(x);
f = (-N/2:N/2-1)*(fs/N);
plot(f,fftshift(abs(y)))
title("spectre Amplitude")

%%
%%suppression du bruit des movements de corps
h = ones(size(x));
fh = 0.5;
index_h = ceil(fh*N/fs);
h(1:index_h) = 0;
h(N-index_h+1:N) = 0;

%%
%TFDI
ecg1_freq = h.*y;
ecg1 = ifft(ecg1_freq,"symmetric");
```

4- Tracer le nouveau signal **ecg1**, et noter les différences par rapport au signal d’origine.

##### Le nouveau signal ecg1
```matlab
plot(t,ecg1);
title("ecg1")
```
![ecg1](https://user-images.githubusercontent.com/121026639/210149873-357a85b5-a7cd-4687-851c-4c1c6db73cca.png)

##### La différence par rapport au signal d'origine
```matlab
plot(t,x-ecg1);
title("La différence")
```
![différence](https://user-images.githubusercontent.com/121026639/210149994-ad1eb265-b493-4f73-8d2a-72b1c97b7dcd.png)

# Suppression-des-interférences-des-lignes-électriques-50Hz

Souvent, l'ECG est contaminé par un bruit du secteur 50 Hz qui doit être supprimé. <br/><br/>

5. Appliquer un filtre Notch idéal pour supprimer cette composante. Les filtres Notch sont utilisés pour rejeter une seule fréquence d'une bande de fréquence
donnée.

```matlab
%suppression de 50 Hz

Notch = ones(size(x));
fcn = 50;
index_hcn = ceil(fcn*N/fs)+1;
Notch(index_hcn) = 0;
Notch(index_hcn+2) = 0;

ecg2_freq = Notch.*fft(ecg1);
ecg2 = ifft(ecg2_freq,"symmetric");
```
6. Visualiser le signal **ecg2** après filtrage.

```matlab
plot(t,ecg2);
title("ecg2")
```
![ecg2](https://user-images.githubusercontent.com/121026639/210153677-e2d5671c-a4c0-4a65-85d0-7f0bf8171d8f.png)

# Amélioration-du-rapport-signal-sur-bruit

Le signal ECG est également atteint par des parasites en provenance de l’activité musculaire extracardiaque du patient. La quantité de bruit est proportionnelle à la
largeur de bande du signal ECG. Une bande passante élevée donnera plus de bruit dans les signaux, et limiter la bande passante peut enlever des détails importants du
signal. <br/><br/>
7. Chercher un compromis sur la fréquence de coupure, qui permettra de préserver la forme du signal ECG et réduire au maximum le bruit. Tester différents choix, puis
tracer et commenter les résultats.  
##### on teste avec 20 Hz

```matlab
pass_bas = zeros(size(x));
fcb = 20;
index_hcb = ceil(fcb*N/fs);
pass_bas(1:index_hcb) = 1;
pass_bas(N-index_hcb+1:N) = 1;

ecg3_freq = pass_bas.*fft(ecg2);
ecg3 = ifft(ecg3_freq,"symmetric");
```
###### On remarque qu'il y a du bruit

![20](https://user-images.githubusercontent.com/121026639/210155249-8be45894-c440-43b6-bfdc-afe0016aa5b4.png)

##### on teste avec 50 Hz

```matlab
pass_bas = zeros(size(x));
fcb = 50;
index_hcb = ceil(fcb*N/fs);
pass_bas(1:index_hcb) = 1;
pass_bas(N-index_hcb+1:N) = 1;

ecg3_freq = pass_bas.*fft(ecg2);
ecg3 = ifft(ecg3_freq,"symmetric");
```

![50](https://user-images.githubusercontent.com/121026639/210155317-490b438b-08e8-43fd-8bb3-6f7524b99b3f.png)

###### On remarque qu'il y a du bruit


##### on teste avec 30 Hz

```matlab
pass_bas = zeros(size(x));
fcb = 30;
index_hcb = ceil(fcb*N/fs);
pass_bas(1:index_hcb) = 1;
pass_bas(N-index_hcb+1:N) = 1;

ecg3_freq = pass_bas.*fft(ecg2);
ecg3 = ifft(ecg3_freq,"symmetric");
```
![30](https://user-images.githubusercontent.com/121026639/210155346-41fb5817-9315-4642-9302-23ff395f0b7f.png)

###### On remarque que le bruit est à-peu-près éliminé

8. Visualiser une période du nouveau signal filtré **ecg3** et identifier autant d'ondes que possible dans ce signal (Voir la partie introduction).

```matlab
plot(t,ecg3);
xlim([0.5 1.5])
title('ecg3')
```

![ecg3](https://user-images.githubusercontent.com/121026639/210155446-05c84601-ea2f-4812-a740-b45a91e807d8.png)

###### On peut identifier les ondes P , QRS et T

# Identification-de-la-fréquence-cardiaque-avec-la-fonction-d’autocorrélation

La fréquence cardiaque peut être identifiée à partir de la fonction d'autocorrélation du signal ECG. Cela se fait en cherchant le premier maximum local après le maximum
global (à tau = 0) de cette fonction.<br /><br/>
9. Ecrire un programme permettant de calculer l’autocorrélation du signal ECG, puis de chercher cette fréquence cardiaque de façon automatique. <br/>  Utiliser ce
programme sur le signal traité ecg3 ou ecg2 et sur le signal ECG non traité. <br/>  **NB** : il faut limiter l’intervalle de recherche à la plage possible de la fréquence cardiaque.

##### ECG3

```matlab
%autocorrélation de signal ECG
[c,lags] = xcorr(ecg3,ecg3);
%stem(lags/fs,c)

E = length(c); %la longueur de la fonction d'autocorrélation
Vector = [0]; %initialisation d'un vecteur
for n = 1:E
    if c(n) > 10
        Vector(end+1) = c(n); %l'ajout de valeur au vecteur
    end
    %pour éliminer les valeurs qui égals au 0
    M = max(Vector);
    in = find(c == M);
    s = lags(in);
    if s <12
        Vector(Vector == M) = [];
    end
end
%extraction de la valeur max de vecteur
Max = max(Vector);
in2 = find(c == Max); %indice de la valeur max
s2 = lags(in2);%la valeur de l'indice dans lags

frequency = (s/fs)*60 %calculer la fréquence
```
###### La fréquence est : 54.8400

 ##### ECG2
 ```matlab
%autocorrélation de signal ECG
[c,lags] = xcorr(ecg2,ecg2);
%stem(lags/fs,c)

E = length(c); %la longueur de la fonction d'autocorrélation
Vector = [0]; %initialisation d'un vecteur
for n = 1:E
    if c(n) > 10
        Vector(end+1) = c(n); %l'ajout de valeur au vecteur
    end
    %pour éliminer les valeurs qui égals au 0
    M = max(Vector);
    in = find(c == M);
    s = lags(in);
    if s <12
        Vector(Vector == M) = [];
    end
end
%extraction de la valeur max de vecteur
Max = max(Vector);
in2 = find(c == Max); %indice de la valeur max
s2 = lags(in2);%la valeur de l'indice dans lags

frequency = (s/fs)*60 %calculer la fréquence
```
###### La fréquence est : 54.7200

10. Votre programme trouve-t-il le bon pouls ? Commenter.

OUI le programme a trouvé le bon pouls




