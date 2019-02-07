(define_automaton "bullet")

;; Bullet has two pipelines, A (Address) and B (Branch).
;; Loads, stores, and FP <-> integer moves use the A-pipe.
;; Branches, MUL/DIV, and FP ops use the B-pipe.
;; Integer ALU ops can use either pipe.

(define_cpu_unit "bullet_A" "bullet")
(define_cpu_unit "bullet_B" "bullet")

(define_cpu_unit "bullet_idiv" "bullet")
(define_cpu_unit "bullet_fpu" "bullet")

(define_insn_reservation "bullet_load" 3
  (and (eq_attr "tune" "bullet")
       (eq_attr "type" "load"))
  "bullet_A")

(define_insn_reservation "bullet_fpload" 2
  (and (eq_attr "tune" "bullet")
       (eq_attr "type" "fpload"))
  "bullet_A")

(define_insn_reservation "bullet_store" 1
  (and (eq_attr "tune" "bullet")
       (eq_attr "type" "store"))
  "bullet_A")

(define_insn_reservation "bullet_fpstore" 1
  (and (eq_attr "tune" "bullet")
       (eq_attr "type" "fpstore"))
  "bullet_A")

(define_insn_reservation "bullet_branch" 1
  (and (eq_attr "tune" "bullet")
       (eq_attr "type" "branch"))
  "bullet_B")

(define_insn_reservation "bullet_sfb_alu" 2
  (and (eq_attr "tune" "bullet")
       (eq_attr "type" "sfb_alu"))
  "bullet_A+bullet_B")

(define_insn_reservation "bullet_jump" 1
  (and (eq_attr "tune" "bullet")
       (eq_attr "type" "jump,call"))
  "bullet_B")

(define_insn_reservation "bullet_mul" 3
  (and (eq_attr "tune" "bullet")
       (eq_attr "type" "imul"))
  "bullet_B")

(define_insn_reservation "bullet_div" 16
  (and (eq_attr "tune" "bullet")
       (eq_attr "type" "idiv"))
  "bullet_B,bullet_idiv*15")

(define_insn_reservation "bullet_alu" 2
  (and (eq_attr "tune" "bullet")
       (eq_attr "type" "unknown,arith,shift,slt,multi,logical,move"))
  "bullet_A|bullet_B")

(define_insn_reservation "bullet_load_immediate" 1
  (and (eq_attr "tune" "bullet")
       (eq_attr "type" "nop,const,auipc"))
  "bullet_A|bullet_B")

(define_insn_reservation "bullet_sfma" 5
  (and (eq_attr "tune" "bullet")
       (and (eq_attr "type" "fadd,fmul,fmadd")
	    (eq_attr "mode" "SF")))
  "bullet_B+bullet_fpu")

(define_insn_reservation "bullet_dfma" 7
  (and (eq_attr "tune" "bullet")
       (and (eq_attr "type" "fadd,fmul,fmadd")
	    (eq_attr "mode" "DF")))
  "bullet_B+bullet_fpu")

(define_insn_reservation "bullet_fp_other" 3
  (and (eq_attr "tune" "bullet")
       (eq_attr "type" "fcvt,fcmp,fmove"))
  "bullet_B+bullet_fpu")

(define_insn_reservation "bullet_fdiv" 16
  (and (eq_attr "tune" "bullet")
       (eq_attr "type" "fdiv,fsqrt"))
  "bullet_B,bullet_fpu*15")

(define_insn_reservation "bullet_i2f" 3
  (and (eq_attr "tune" "bullet")
       (eq_attr "type" "mtc"))
  "bullet_A")

(define_insn_reservation "bullet_f2i" 3
  (and (eq_attr "tune" "bullet")
       (eq_attr "type" "mfc"))
  "bullet_A")

(define_bypass 1 "bullet_load,bullet_alu,bullet_mul,bullet_f2i,bullet_sfb_alu"
  "bullet_alu,bullet_branch")

(define_bypass 1 "bullet_alu,bullet_sfb_alu"
  "bullet_sfb_alu")

(define_bypass 1 "bullet_load,bullet_alu,bullet_mul,bullet_f2i,bullet_sfb_alu"
  "bullet_store" "store_data_bypass_p")

(define_bypass 2 "bullet_i2f"
  "bullet_sfma,bullet_dfma,bullet_fp_other,bullet_fdiv")

(define_bypass 2 "bullet_fp_other"
  "bullet_sfma,bullet_dfma,bullet_fp_other,bullet_fdiv")

(define_bypass 2 "bullet_fp_other"
  "bullet_alu,bullet_branch")

(define_bypass 2 "bullet_fp_other"
  "bullet_store" "store_data_bypass_p")
