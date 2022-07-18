;; Constraint definitions for RISC-V target.
;; Copyright (C) 2011-2022 Free Software Foundation, Inc.
;; Contributed by Andrew Waterman (andrew@sifive.com).
;; Based on MIPS target for GNU compiler.
;;
;; This file is part of GCC.
;;
;; GCC is free software; you can redistribute it and/or modify
;; it under the terms of the GNU General Public License as published by
;; the Free Software Foundation; either version 3, or (at your option)
;; any later version.
;;
;; GCC is distributed in the hope that it will be useful,
;; but WITHOUT ANY WARRANTY; without even the implied warranty of
;; MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
;; GNU General Public License for more details.
;;
;; You should have received a copy of the GNU General Public License
;; along with GCC; see the file COPYING3.  If not see
;; <http://www.gnu.org/licenses/>.

;; Register constraints

(define_register_constraint "f" "TARGET_HARD_FLOAT ? FP_REGS : NO_REGS"
  "A floating-point register (if available).")

(define_register_constraint "j" "SIBCALL_REGS"
  "@internal")

;; Avoid using register t0 for JALR's argument, because for some
;; microarchitectures that is a return-address stack hint.
(define_register_constraint "l" "JALR_REGS"
  "@internal")

;; General constraints

(define_constraint "I"
  "An I-type 12-bit signed immediate."
  (and (match_code "const_int")
       (match_test "SMALL_OPERAND (ival)")))

(define_constraint "J"
  "Integer zero."
  (and (match_code "const_int")
       (match_test "ival == 0")))

(define_constraint "K"
  "A 5-bit unsigned immediate for CSR access instructions."
  (and (match_code "const_int")
       (match_test "IN_RANGE (ival, 0, 31)")))

(define_constraint "L"
  "A U-type 20-bit signed immediate."
  (and (match_code "const_int")
       (match_test "LUI_OPERAND (ival)")))

;; Floating-point constant +0.0, used for FCVT-based moves when FMV is
;; not available in RV32.
(define_constraint "G"
  "@internal"
  (and (match_code "const_double")
       (match_test "op == CONST0_RTX (mode)")))

(define_memory_constraint "A"
  "An address that is held in a general-purpose register."
  (and (match_code "mem")
       (match_test "GET_CODE(XEXP(op,0)) == REG")))

(define_constraint "S"
  "A constraint that matches an absolute symbolic address."
  (match_operand 0 "absolute_symbolic_operand"))

(define_constraint "U"
  "@internal
   A PLT-indirect call address."
  (match_operand 0 "plt_symbolic_operand"))

(define_constraint "T"
  "@internal
   A constant @code{move_operand}."
  (and (match_operand 0 "move_operand")
       (match_test "CONSTANT_P (op)")))

(define_predicate "imm2u_operand"
  (and (match_operand 0 "const_int_operand")
       (match_test "satisfies_constraint_u02 (op)")))

(define_predicate "imm3u_operand"
  (and (match_operand 0 "const_int_operand")
       (match_test "satisfies_constraint_u03 (op)")))

(define_predicate "imm4u_operand"
  (and (match_operand 0 "const_int_operand")
       (match_test "satisfies_constraint_u04 (op)")))

(define_predicate "imm5u_operand"
  (and (match_operand 0 "const_int_operand")
       (match_test "satisfies_constraint_u05 (op)")))

(define_predicate "imm6u_operand"
  (and (match_operand 0 "const_int_operand")
       (match_test "satisfies_constraint_u06 (op)")))

(define_predicate "rimm3u_operand"
  (ior (match_operand 0 "register_operand")
       (match_operand 0 "imm3u_operand")))

(define_predicate "rimm4u_operand"
  (ior (match_operand 0 "register_operand")
       (match_operand 0 "imm4u_operand")))

(define_predicate "rimm5u_operand"
  (ior (match_operand 0 "register_operand")
       (match_operand 0 "imm5u_operand")))

(define_predicate "rimm6u_operand"
  (ior (match_operand 0 "register_operand")
       (match_operand 0 "imm6u_operand")))

(define_predicate "const_insb64_operand"
  (and (match_code "const_int")
       (match_test "IN_RANGE (INTVAL (op), 0, 7)")))

(define_predicate "imm_1_2_4_8_operand"
  (and (match_operand 0 "const_int_operand")
       (ior (ior (match_test "satisfies_constraint_C01 (op)")
		 (match_test "satisfies_constraint_C02 (op)"))
	    (ior (match_test "satisfies_constraint_C04 (op)")
		 (match_test "satisfies_constraint_C08 (op)")))))

(define_predicate "pwr_7_operand"
  (and (match_code "const_int")
       (match_test "INTVAL (op) != 0
		    && (unsigned) exact_log2 (INTVAL (op)) <= 7")))

(define_predicate "imm_0_1_operand"
  (and (match_operand 0 "const_int_operand")
       (ior (match_test "satisfies_constraint_C00 (op)")
	    (match_test "satisfies_constraint_C01 (op)"))))

(define_predicate "imm_1_2_operand"
  (and (match_operand 0 "const_int_operand")
       (ior (match_test "satisfies_constraint_C01 (op)")
	    (match_test "satisfies_constraint_C02 (op)"))))

(define_predicate "imm_2_3_operand"
  (and (match_operand 0 "const_int_operand")
       (ior (match_test "satisfies_constraint_C02 (op)")
	    (match_test "satisfies_constraint_C03 (op)"))))

(define_predicate "imm_15_16_operand"
  (and (match_operand 0 "const_int_operand")
       (ior (match_test "satisfies_constraint_C15 (op)")
	    (match_test "satisfies_constraint_C16 (op)"))))

(define_predicate "rev_rimm_operand"
  (ior (match_operand 0 "const_arith_operand")
       (match_test "INTVAL (op) == (BITS_PER_WORD - 1)")))

(define_predicate "fsr_shamt_imm"
  (ior (match_operand 0 "register_operand")
       (and (match_operand 0 "const_arith_operand")
            (match_test "IN_RANGE (INTVAL (op), 1, 31)"))))
