# Programa-Gravar-db
Este programa utiliza os parâmetros de permissividade de permeabilidade para estimar a refletividade de um material.
from __future__ import division
import matplotlib.pyplot as plt
import numpy as np
import scipy.integrate as si
from matplotlib.widgets import Slider, Button, RadioButtons
#from matplotlib.ticker import MultipleLocator
import os

"""
1-) Ler S11 e S21 do arquivo txt
2-) Aplicar o NRW para encontrar e and u
3-) Utilizar a Fórmula do Zin
4-) Variar o d, apenas para fórmula do Zin
"""

#LER ARQUIVO TXT NA PASTA
TXT =[]

for arquivo in os.listdir('.'):
	if arquivo[len(arquivo)-4:] == '.txt':
		TXT.append(arquivo)
		
print '\nArquivo: ',TXT[0]	
	
arq = open(TXT[0],'r')
ler = arq.readlines()
arq.close()

#ESCOLHA A BANDA-------------------
#BANDA-X
a = 22.86e-3 #[m] Base maior do guia de onda (X-Band)
offset = 9.76e-3 #Espessura do 1/4 de lambda (X- Band)
freq_corte = 6.56e9 #[Hz] Frequência de Corte (X-Band)

"""
# BANDA-Ku
a = 15.80e-3 #[m] Base maior do guia de onda (Ku-Band)
offset = 6.50e-3 #Espessura do 1/4 de lambda (Ku- Band)
freq_corte = 9.49e9 #[Hz] Frequência de Corte (Ku-Band)
"""
#---------------------------------



#Ajuste da Referencia de L1 e L2
d = 9.78e-3 #[m] #Espessura da amostra (Livro chama de L)
L1 = 0e-3 #[m] #Plano de referencia porta 1
L2 = offset - d #[m] #Plano de referencia porta 2


#CONSTANTES
c =2.998e8 #[m/s] #velocidade da Luz no vacuo
u0=4*np.pi*1e-7 # permeabilidade do vacuo
onda_cut= c/freq_corte #[m] #lambda de corte

#Vetores - 1	
F=[] #frequencia [Hz] DE CALCULO
F_grafic=[] #FREQUENCIA PARA PLOTAR EM [GHz]

s11r=[] #real
s11i=[] #imag
s21r=[] #real
s21i=[] #imag

s11=[] # real + j imag
s21=[] # real + j imag

s11c=[] # real + j imag (Corrigido)
s21c=[] # real + j imag (Corrigido)

#ORGANIZAR PARAMETROS-S em VETOR
ler1_col=1 #S11r
ler2_col=2 #S11i
ler3_col=3 #S21r #VNA PEDRO
ler4_col=4 #S21i #VNA PEDRO
#ler3_col=3 #S21r #VNA NEWTON
#ler4_col=4 #S21i #VNA NEWTON


#Vetores - 2	
er_r = [] #permissividade real
er_i = [] #permissividade imag
ur_r = [] #permeabilidade real
ur_i = [] #permeabilidade imag

er_abs=[] #permissividade modulo
ur_abs=[] #permeabilidade modulo

EX = []
UX = []

#Vetroes - 3
s11_v =[] #[a.u]


os.chdir("./dados_Teoricos")

valor = [2e-3]

#for d in np.arange(1.5e-3,3.1e-3,0.1e-3):
for d in valor:

	s11_v =[]
	F= []

	for n in range(0,len(ler)):
	
	
		dados = ler[n].split(',')	
		#frequencia
		f = float(dados[0])*1e9
		F.append(f)
		F_grafic.append(f/1e9)
		
		#Permissividade NRW
		ex = float(dados[1]) + 1j*float(dados[2])
		er_r.append(ex.real)
		er_i.append(ex.imag)
		EX.append(ex)

				
		#Permeabilidade NRW
		ux = float(dados[3]) + 1j*float(dados[4])
		ur_r.append(ux.real)
		ur_i.append(ux.imag)
		UX.append(ux)	
				

			
		#*********************Cálculo da Refletividade (RL)***************
		
		#Calcular impedância de entrada
		#z = (50j*(UX[n]/EX[n])**(1.0/2.0))*np.tan(1*(2*np.pi*d*f/c)*((UX[n]*EX[n])**(1.0/2.0)))
		z = (50*(UX[n]/EX[n])**(1.0/2.0))*np.tanh(1j*(2*np.pi*d*f/c)*((UX[n]*EX[n])**(1.0/2.0))) #COM TANH
		
		#Passar para S11(dB)
		db= -20*np.log10(abs((z-50)/(z+50))) #dB #somente para voltagem
		#db = -20*np.log10(abs((z-50)/(z+50))) #dB para potência
		
		#Passar para Lienar Mag S11 (%)
		s11_curto = (10.0**((db/10.0))) # a.u
		

		#s11_v.append(s11_curto*100)# LINEAR
		s11_v.append(round(db,5)) # dB
	#Gravar
	espessura = round(d/1e-3,2)
	grav = open('./'+TXT[0][:-4]+"_"+str(espessura)+"mm.txt",'w')
	titulo = "%10s	%s\n"%("Freq (Hz)", "RL(dB)")
	grav.write(titulo)
	
	for i in range(0,len(F)):
		
		
		escrever = "%.2f	%.2f\n"%(F[i],s11_v[i])
		
		grav.write(escrever)
	grav.close()
