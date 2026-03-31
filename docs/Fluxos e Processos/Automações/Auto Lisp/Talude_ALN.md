;;;========================================================================
;;; COMANDO: TALUDEALN
;;; DESCRIÇÃO: Gera linhas transversais de talude mantendo paralelismo.
;;;========================================================================

(defun c:TALUDEALN ( / entTop entBot dist spc lenTop curDist ptTop ptBot ptMid isShort undoObj doc vlaBot angBase ptFar1 ptFar2 tmpLine intPts)
  ;; Carrega as funções do Visual LISP
  (vl-load-com)
  
  ;; Configura o Undo para poder desfazer tudo com um único Ctrl+Z
  (setq doc (vla-get-ActiveDocument (vlax-get-acad-object)))
  (vla-StartUndoMark doc)

  ;; Solicita as linhas ao usuário
  (setq entTop (car (entsel "\nSelecione a linha da CRISTA (topo do talude): ")))
  (if (not entTop) (progn (princ "\nCrista não selecionada. Comando cancelado.") (exit)))

  (setq entBot (car (entsel "\nSelecione a linha do PÉ (base do talude): ")))
  (if (not entBot) (progn (princ "\nPé não selecionado. Comando cancelado.") (exit)))

  ;; Solicita o espaçamento
  (setq spc (getreal "\nInforme o espaçamento entre as linhas <2.0>: "))
  (if (not spc) (setq spc 2.0)) ; Valor padrão

  ;; NOVIDADE: Pede o ângulo de alinhamento para forçar o paralelismo
  (setq angBase (getangle "\nInforme o ângulo de alinhamento das linhas <Pressione Enter para Vertical>: "))
  (if (not angBase) (setq angBase (/ pi 2.0))) ; Padrão é 90 graus (pi/2 radianos)

  ;; Obtém o comprimento total da linha da Crista
  (setq lenTop (vlax-curve-getDistAtParam entTop (vlax-curve-getEndParam entTop)))
  (setq curDist 0.0)
  (setq isShort nil) 

  ;; Converte a entidade do pé para objeto VLA para usar a função IntersectWith
  (setq vlaBot (vlax-ename->vla-object entBot))

  ;; Loop de criação das linhas
  (while (<= curDist lenTop)
    ;; Pega o ponto na crista com base na distância atual
    (setq ptTop (vlax-curve-getPointAtDist entTop curDist))
    
    ;; Cria pontos distantes usando o ângulo fixo para traçar uma reta infinita
    (setq ptFar1 (polar ptTop angBase 100000.0))
    (setq ptFar2 (polar ptTop (+ angBase pi) 100000.0))

    ;; Cria uma linha temporária invisível que cruza o desenho no ângulo escolhido
    (entmake (list '(0 . "LINE") (cons 10 ptFar1) (cons 11 ptFar2)))
    (setq tmpLine (vlax-ename->vla-object (entlast)))

    ;; Encontra a interseção matemática dessa linha direcional com a linha do PÉ
    (setq intPts (vlax-invoke vlaBot 'IntersectWith tmpLine 0)) ; 0 = acExtendNone
    
    ;; Deleta a linha temporária
    (vla-delete tmpLine)

    ;; Se houver interseção, desenha a linha do talude
    (if intPts
      (progn
        ;; Pega as coordenadas X, Y, Z do primeiro ponto de interseção
        (setq ptBot (list (car intPts) (cadr intPts) (caddr intPts)))

        (if isShort
          ;; Desenha linha CURTA (50% do comprimento)
          (progn
            (setq ptMid (list
                          (+ (car ptTop) (* 0.5 (- (car ptBot) (car ptTop))))
                          (+ (cadr ptTop) (* 0.5 (- (cadr ptBot) (cadr ptTop))))
                          (+ (caddr ptTop) (* 0.5 (- (caddr ptBot) (caddr ptTop))))
                        ))
            (entmake (list '(0 . "LINE") (cons 10 ptTop) (cons 11 ptMid)))
            (setq isShort nil) 
          )
          ;; Desenha linha LONGA (100% do comprimento)
          (progn
            (entmake (list '(0 . "LINE") (cons 10 ptTop) (cons 11 ptBot)))
            (setq isShort T) 
          )
        )
      )
    )
    
    ;; Avança para o próximo ponto
    (setq curDist (+ curDist spc))
  )

  ;; Finaliza o Undo
  (vla-EndUndoMark doc)
  
  (princ "\nRepresentação do talude gerada com sucesso e alinhada!")
  (princ)
)