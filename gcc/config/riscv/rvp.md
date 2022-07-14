;; Machine description for P extension.
;; Copyright (C) 2021 Free Software Foundation, Inc.

;; This file is part of GCC.

;; GCC is free software; you can redistribute it and/or modify
;; it under the terms of the GNU General Public License as published by
;; the Free Software Foundation; either version 3, or (at your option)
;; any later version.

;; GCC is distributed in the hope that it will be useful,
;; but WITHOUT ANY WARRANTY; without even the implied warranty of
;; MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
;; GNU General Public License for more details.

;; You should have received a copy of the GNU General Public License
;; along with GCC; see the file COPYING3.  If not see
;; <http://www.gnu.org/licenses/>.



;; cmix
(define_insn "cmix<X:mode>"
  [(set (match_operand:X 0 "register_operand"       "=r")
	  (ior:X
	    (and:X
	      (match_operand:X 1 "register_operand" " r")
	      (match_operand:X 3 "register_operand" " r"))
	    (and:X
	      (match_operand:X 2 "register_operand" " r")
	      (not:X (match_dup 3)))))]
  "TARGET_ZBPBO"
  "cmix\t%0, %3, %1, %2"
  [(set_attr "type"   "dsp")
   (set_attr "mode"   "<MODE>")])

(define_insn "clzsi2_rvp"
  [(set (match_operand:SI 0 "register_operand" "=r")
        (clz:SI (match_operand:SI 1 "register_operand" "r")))]
  "TARGET_ZBPBO && !TARGET_64BIT"
  "clz\t%0, %1"
  [(set_attr "type" "simd")
   (set_attr "mode" "SI")])

;; MAX, MIN
(define_insn "smax<mode>3_rvp"
  [(set (match_operand:X 0 "register_operand"          "=r")
	(smax:X (match_operand:X 1 "register_operand"  " r")
		 (match_operand:X 2 "register_operand" " r")))]
  "TARGET_ZBPBO"
  "max\t%0, %1, %2"
  [(set_attr "type" "dsp")
   (set_attr "mode" "<MODE>")])

(define_insn "smin<mode>3_rvp"
  [(set (match_operand:X 0 "register_operand"          "=r")
	(smin:X (match_operand:X 1 "register_operand"  " r")
		 (match_operand:X 2 "register_operand" " r")))]
  "TARGET_ZBPBO"
  "min\t%0, %1, %2"
  [(set_attr "type" "dsp")
   (set_attr "mode" "<MODE>")])

;; rev
(define_expand "rev<mode>"
  [(match_operand:X 0 "register_operand")
   (match_operand:X 1 "register_operand")]
  "TARGET_ZBPBO"
{
  emit_insn (gen_rev<mode>_internal (operands[0], operands[1]));
  DONE;
})

(define_insn "revsi_internal"
  [(set (match_operand:SI 0 "register_operand"             "=r")
	(unspec:SI [(match_operand:SI 1 "register_operand" " r")
		    (const_int 31)] UNSPEC_BITREV))]
  "TARGET_ZBPBO && !TARGET_64BIT"
  "rev\t%0, %1"
  [(set_attr "type"   "dsp")
   (set_attr "mode"   "SI")])

(define_insn "revdi_internal"
  [(set (match_operand:DI 0 "register_operand"             "=r")
	(unspec:DI [(match_operand:DI 1 "register_operand" " r")
		    (const_int 63)] UNSPEC_BITREV))]
  "TARGET_ZBPBO && TARGET_64BIT"
  "rev\t%0, %1"
  [(set_attr "type"   "dsp")
   (set_attr "mode"   "DI")])

;; swap8
(define_insn "bswap8<mode>"
  [(set (match_operand:X 0 "register_operand" "=r")
	(unspec:X [(match_operand:X 1 "register_operand" "r")] UNSPEC_BSWAP))]
  "TARGET_ZBPBO"
  "rev8.h\t%0, %1"
  [(set_attr "type"  "dsp")
   (set_attr "mode"  "<MODE>")])

;; fsr, fsri, fsrw
(define_insn "fsrw"
  [(set (match_operand:SI 0 "register_operand"     "=r")
	(unspec: SI [(match_operand:SI 1 "register_operand" "r")
		(match_operand:SI 2 "register_operand" "r")
		(match_operand:SI 3 "register_operand" "r")] UNSPEC_FSRW))]
  "TARGET_ZBPBO && TARGET_64BIT"
  "fsrw\t%0,%1,%2,%3"
  [(set_attr "type" "dsp")
   (set_attr "mode" "SI")])

(define_expand "fsr"
  [(match_operand:SI 0 "register_operand" "  =r, r")
   (match_operand:SI 1 "register_operand" "r, r")
   (match_operand:SI 2 "arith_operand" "r, I")
   (match_operand:SI 3 "register_operand" "r, r")]
   "TARGET_ZBPBO && !TARGET_64BIT"
  {
    unsigned HOST_WIDE_INT shamt;
    if (CONST_INT_P (operands[2]))
      {
        shamt = INTVAL (operands[2]) & 63;
        if (shamt == 32)
		  {
            emit_move_insn (operands[0], operands[1]);
            DONE;
          }
        shamt = shamt > 32 ? shamt - 32 : shamt;
        operands[2] = GEN_INT(shamt);
        emit_insn (gen_fsri_rvp (operands[0], operands[1],
                       operands[2], operands[3]));
      }
	else
	  {
		emit_insn (gen_fsr_rvp (operands[0], operands[1], operands[2], operands[3]));
	  }
	DONE;
  })

(define_insn "fsr_rvp"
  [(set (match_operand:SI 0 "register_operand"     "=r")
	(unspec: SI [(match_operand:SI 1 "register_operand" "")
		(match_operand:SI 2 "register_operand" "")
		(match_operand:SI 3 "register_operand" "")] UNSPEC_FSR))]
  "TARGET_ZBPBO && !TARGET_64BIT"
  "fsr\t%0,%1,%2,%3"
  [(set_attr "type" "dsp")
   (set_attr "mode" "SI")])

(define_insn "fsri_rvp"
  [(set (match_operand:SI 0 "register_operand"     "=r")
	(truncate: SI
	  (ior:DI
	    (ashiftrt:DI
	      (match_operand:SI 1 "register_operand" " r")
	      (match_operand:SI 2 "fsr_shamt_imm"   " u05"))
	    (lshiftrt:DI
	      (match_operand:SI 3 "register_operand" " r")
	      (minus:SI (const_int 32) (match_dup 2))))))]
  "TARGET_ZBPBO && !TARGET_64BIT"
  "fsri\t%0,%1,%2,%3"
  [(set_attr "type" "dsp")
   (set_attr "mode" "SI")])

;; Pack (wait for zpn)


