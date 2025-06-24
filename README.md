# algerian-cafe-sim
"use client"

import { Canvas } from "@react-three/fiber"
import { OrbitControls, useGLTF } from "@react-three/drei"
import * as THREE from "three"
import { useState, useRef } from "react"

// مكون زبون يحمل نموذج 3D
function CustomerModel({
  modelPath,
  initialPosition,
  scale = 1,
}: {
  modelPath: string
  initialPosition: [number, number, number]
  scale?: number
}) {
  const { scene } = useGLTF(modelPath)
  const meshRef = useRef<THREE.Group>(null)
  const [position, setPosition] = useState(new THREE.Vector3(...initialPosition))
  const [target, setTarget] = useState(
    new THREE.Vector3(
      Math.random() * 4 - 2,
      initialPosition[1],
      Math.random() * 4 - 2
    )
  )

  // حركة بسيطة للزبون: يمشي بين نقاط عشوائية
  useFrame(() => {
    if (!meshRef.current) return
    const dir = target.clone().sub(position)
    const dist = dir.length()
    if (dist < 0.05) {
      setTarget(
        new THREE.Vector3(
          Math.random() * 4 - 2,
          initialPosition[1],
          Math.random() * 4 - 2
        )
      )
    } else {
      dir.normalize()
      position.add(dir.multiplyScalar(0.01))
      setPosition(position.clone())
      meshRef.current.position.copy(position)
    }
  })

  return <primitive ref={meshRef} object={scene} scale={scale} position={position} />
}

// المشهد الرئيسي للعبة المقهى
export default function CafeScene() {
  return (
    <div className="w-full h-screen">
      <Canvas camera={{ position: [3, 3, 5], fov: 50 }}>
        <ambientLight intensity={0.5} />
        <directionalLight position={[5, 5, 5]} intensity={1} />
        <OrbitControls />

        {/* الأرضية */}
        <mesh rotation={[-Math.PI / 2, 0, 0]} position={[0, -0.5, 0]}>
          <planeGeometry args={[20, 20]} />
          <meshStandardMaterial color="#c2b280" />
        </mesh>

        {/* الطاولة */}
        <mesh position={[0, 0, 0]}>
          <cylinderGeometry args={[0.5, 0.5, 0.1, 32]} />
          <meshStandardMaterial color="#8B4513" />
        </mesh>

        {/* كرسي */}
        <mesh position={[1, 0, 0]}>
          <boxGeometry args={[0.4, 0.4, 0.4]} />
          <meshStandardMaterial color="#a0522d" />
        </mesh>

        {/* الزبائن بنماذج 3D */}
        <CustomerModel modelPath="/models/old_man.glb" initialPosition={[1, 0, -1]} scale={1} />
        <CustomerModel modelPath="/models/young_man.glb" initialPosition={[-1, 0, 0]} scale={1} />
        <CustomerModel modelPath="/models/child.glb" initialPosition={[0, 0, 1]} scale={0.7} />
      </Canvas>
    </div>
  )
}
