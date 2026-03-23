---
title: Interpolador de Cotas com AutoLISP
description: Script rápido para calcular cotas topográficas no CAD.
tags: [autocad, lisp, topografia]
---

# Interpolação Rápida de Cotas (AutoLISP)

  **Objetivo:** Automatizar a interpolação entre dois pontos topográficos extraindo valores direto dos atributos dos blocos.  
    
  **Última Atualização:** 22/03/2026

## 1. Aplicação e Pré-requisitos
* **Contexto:** Agilizar a obtenção de cotas topográficas em projetos de infraestrutura.
* **Ferramenta:** AutoCAD ou ZWCAD.
* **Requisito:** A base topográfica precisa ter as cotas armazenadas em Blocos com Atributos numéricos.

## 2. Como Usar (Passo a Passo)
* **Inicie o comando:** Digite `INTERPOLA_COTA` no prompt.
* **Referência 1:** Clique no bloco da cota menor.
* **Referência 2:** Clique no bloco da cota maior.
* **Interpolação:** Uma linha guia aparecerá. Clique no ponto intermediário desejado. O script calcula a declividade automaticamente e exibe a cota exata do ponto clicado diretamente na linha de comando.

## 3. Código Lisp

``` Lisp
(defun c:INTERPOLA_COTA (/ data1 data2 pMenor pMaior p3 valMenor valMaior diffLevel distTotal declividade distParcial resultado txtResultado)
  (vl-load-com)
  (setvar "cmdecho" 0)
  
  (defun GetBlockData (msg / ent elist pt en strVal dotStr valNum cota)
    (setq ent (car (entsel msg)))
    (if (and ent (= (cdr (assoc 0 (entget ent))) "INSERT"))
      (progn
        (setq elist (entget ent))
        (setq pt (cdr (assoc 10 elist)))
        (setq en (entnext ent))
        (setq cota nil)
        (while (and en (= (cdr (assoc 0 (entget en))) "ATTRIB") (not cota))
          (setq strVal (cdr (assoc 1 (entget en))))
          (setq valNum (distof strVal))
          (if (not valNum)
            (progn
              (setq dotStr (vl-string-translate "," "." strVal))
              (setq valNum (distof dotStr))
            )
          )
          (if valNum (setq cota valNum))
          (setq en (entnext en))
        )
        (if cota (list pt cota) (progn (princ "\nErro: Sem valor numérico.") nil))
      )
      (progn (princ "\nObjeto inválido. Selecione um bloco com atributo.") nil)
    )
  )
  
  (princ "\n--- INTERPOLAÇÃO DE COTAS ---")
  (setq data1 (GetBlockData "\n>>> 1. Selecione o bloco da COTA MENOR: "))
  (if data1
    (progn
      (setq data2 (GetBlockData "\n>>> 2. Selecione o bloco da COTA MAIOR: "))
      (if data2
        (progn
          (setq pMenor (car data1) valMenor (cadr data1))
          (setq pMaior (car data2) valMaior (cadr data2))
          (setq distTotal (distance pMenor pMaior))
          
          (if (> distTotal 0.001)
            (progn
              (setq declividade (/ (- valMaior valMenor) distTotal))
              (princ (strcat "\n[Dados] Distância: " (rtos distTotal 2 3) " | Declividade: " (rtos declividade 2 6)))
              
              (while (setq p3 (getpoint pMaior "\n>>> 3. Clique no ponto para obter a cota (ou ESC para sair): "))
                (setq distParcial (distance pMaior p3))
                (setq resultado (- valMaior (* distParcial declividade)))
                (setq txtResultado (vl-string-translate "." "," (rtos resultado 2 2)))
                (princ (strcat "\n>>> COTA INTERPOLADA: " txtResultado " <<<"))
              )
            )
            (princ "\nErro: Distância zero.")
          )
        )
      )
    )
  )
  (setvar "cmdecho" 1)
  (princ)
)
```
  
  
**Vídeo do script sendo executado.**

<video width="100%" controls muted>
  <source src="/assets/23-13-41.mp4" type="video/mp4">
</video>