# -*- coding: utf-8 -*-
"""
Created on Sat Mar 13 17:31:48 2021

@author: quent
"""


#On possède un jeu de valeurs associées à la température d'une étoile
#on pense que la courbe de température de cette étoile suit une 
#fonction de Weierstrass : t(i) = sum(0,c,a^n*cos(b^n*pi*i))
#t(i) la température en fonction de l'instant i
#le but est de retrouver les paramètres a,b et c qui forment la 
#fctn de W qui correspondent le mieux aux échantillons
#L'individu est le triplet de paramètres a,b,c
#La fonction de fitness doit comparer la sortie de la fcn de W créée
#aux elem de l'échantillon
#Pour introduire des mutations, on peut changer la valeur d'un ou 
#tous les param toutes les 3 générations




#Import des librairies
import numpy as np
import timeit
import scipy as sp
import math as m
import matplotlib.pyplot as plt
import random
import keyboard as kb
from math import cos
from math import pi 

#0.35    15    3
#0.49    15    12

#t(i) = sum(0,c,a^n*cos(b^n*pi*i))
def W(i):
    y=0
    for n in range(c):        
        y = y + a**n*cos(b**n*pi*i)
    return y
def W2(i,ind):
    y=0
    for n in range(ind[2]):        
        y = y + ind[0]**n*cos(ind[1]**n*pi*i)
    return y
i = np.linspace(0,10,1000)
out = np.vectorize(W)




with open ("temperature_sample_calibrate2.csv","r") as file:
    temps = []
    temperature = []
    next(file)
    for line in file:
        line2 = line.split(";")
        line2 = [float(i) for i in line2]
        temps.append(line2[0])
        temperature.append(line2[1])
#plt.plot(temps,temperature)

with open ("temperature_sample.csv","r") as file:
    temps = []
    temperature = []
    next(file)
    for line in file:
        line2 = line.split(";")
        line2 = [float(i) for i in line2]
        temps.append(line2[0])
        temperature.append(line2[1])
plt.plot(temps,temperature)

def createRandomCoeff(n):
    liste = []
    for i in range(n) :
        a = random.uniform(0,1)#il faudra enlever le round !!!!!
        b = random.randint(1,20)
        c = random.randint(1,20)
        ind = [a,b,c]
        liste.append(ind)        
    return liste
def afficheInd(ind):
    print(str(round(ind[0],2))+"    "+ str(ind[1]) +"    "+ str(ind[2]))   
def affichePop(pop):
    for i in pop:
        afficheInd(i)
def fitness(ind):
    fitTot = 0
    for i in range(len(temps)):
        fit = abs(W2(temps[i],ind)-temperature[i])
        fitTot += fit
    return fitTot
def evaluate(pop):
    return sorted(pop, key = lambda x: fitness(x))
def selection(pop, hcount, lcount):
    liste = pop[:hcount].copy()+ pop[len(pop)-lcount:len(pop)].copy()
    return liste
def croisement(ind1,ind2):
    ind3 = ind1.copy()
    ind3[0]  = ind2[0]
    ind4 = ind2
    ind4[0]  = ind1[0]
    
    ind5 = ind1.copy()
    ind5[1]  = ind2[1]
    ind6 = ind2
    ind6[1]  = ind1[1]
    
    ind7 = ind1.copy()
    ind7[2]  = ind2[2]
    ind8 = ind2
    ind8[2]  = ind1[2]
    return [ind3,ind4,ind5,ind6,ind7,ind8]
def mutation(ind):
    param = random.randint(0,2)
    a = random.uniform(0,1)
    b = random.randint(1,20)
    c = random.randint(1,20)
    if(param == 0):
        ind[0] = a
    elif(param == 1):
        ind[1] = b
    else:
        ind[2] = c
    return ind
bestListe = []

def loop():
    pop = createRandomCoeff(30)
    iterationNbre = 0
    foundSol = False
    while foundSol == False :
        popTrie = evaluate(pop)
        if(fitness(popTrie[0])<0.75):
            print()
            print("Solution trouvée !")
            print()
            print("Generation = " + str(iterationNbre))
            afficheInd(popTrie[0])
            print("Fitness = " + str(fitness(popTrie[0])))
            foundSol = True
        else:
            iterationNbre +=1
            popSelect = selection(popTrie,20,4)
            popCroise = []
            for i in range(0,len(popSelect)-1,2):
                popCroise += croisement(popSelect[i], popSelect[i+1])
            popMute = []
            for i in popSelect:
                popMute.append(mutation(i))
            newPop = createRandomCoeff(4)
            endPop = popSelect + popCroise + popMute + newPop
            finalPop = evaluate(endPop)
            bestInd = finalPop[0]
            bestListe.append(bestInd)
            popTrie = finalPop
            print("Generation = " + str(iterationNbre))
            afficheInd(bestInd)
            print("Fitness = " + str(fitness(finalPop[0])))

loop()
print()
shortBest = bestListe[len(bestListe)-10:len(bestListe)]
print("Voici les 10 meilleurs résultats(ordre décroissant)")
print("A       B    C")
affichePop(shortBest)
#attention la liste est inversée
bestInd = bestListe[len(bestListe)-1]
bestFitness = fitness(bestInd)
a = bestInd[0]
b = bestInd[1]
c = bestInd[2]


plt.plot(i,out(i))

plt.annotate('a = ' + str(a),xy=(8,1), fontsize=20)
plt.annotate('b = ' + str(b),xy=(8,0.7), fontsize=20)
plt.annotate('c = ' + str(c),xy=(8,0.4), fontsize=20)
#plt.annotate('Fitness = ' + str(round(bestFitness,2)),xy=(6,0.1), fontsize=20)









