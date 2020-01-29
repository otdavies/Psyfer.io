---
categories: ["unity", "c#", "led", "3d printing"]
date: "2015-11-05T17:52:41-05:00"
title: "C# Post Test"
description: "Some very long description here blah blah blah blah blah blah blah blah blah blah blah blah blah blah blah blah blah blah blah blah blah blah blah blah blah blah blah blah blah blah blah blah blah blah blah blah"
---

What does some good ol' c# look like on here?

## C# incoming


``` csharp

using System.Runtime.InteropServices;
using UnityEngine;

[RequireComponent(typeof(MeshRenderer))]
public class HighDetailGraph : MonoBehaviour
{
    // Inspector fields
    [SerializeField]
    private int _vertexBudget = 1024;
    [SerializeField]
    private int _lodLevels = 4;

    // Internal fields
    private MeshFilter _meshFilter;
    private MeshRenderer _meshRenderer;
    private Material _graphMaterial;
    private ComputeBuffer _graphBuffer;
    private Mesh[] _lods;

    private Datum[] _fakeData;

    [StructLayout(LayoutKind.Sequential)]
    private struct Datum
    {
        public const int size = sizeof(float) * 6 + sizeof(float);
        public Vector3 position;
        public Vector3 tangent;
        public float   data;

        public Datum(Vector3 position, Vector3 tangent, float data)
        {
            this.position = position;
            this.tangent = tangent;
            this.data = data;
        }
    }

    private void OnEnable()
    {
        _fakeData = new Datum[_vertexBudget];
        _meshFilter = GetComponent<MeshFilter>();
        _meshRenderer = GetComponent<MeshRenderer>();
        _graphMaterial = _meshRenderer.material;
        _lods = GenerateLODMeshes();

        _meshFilter.mesh = _lods[0];
        _meshRenderer.allowOcclusionWhenDynamic = false;

        for (int i = 0; i < _fakeData.Length; i++)
        {
            _fakeData[i] = new Datum(new Vector3(0, i * 0.5f, 0), Vector3.up, Mathf.PerlinNoise(i * 0.1f, i * 0.3f) * -10);
        }

        _graphBuffer = new ComputeBuffer(_fakeData.Length, Datum.size);
        _graphBuffer.SetData(_fakeData);
        _graphMaterial.SetBuffer("_Data", _graphBuffer);
    }

    private void OnDisable()
    {
        _graphBuffer.Dispose();
    }

    int lodLevel = 0;

    private void Update()
    {
        if (Input.GetKeyDown(KeyCode.Space))
        {
            _meshFilter.mesh = _lods[++lodLevel];
        }
    }

    private Mesh[] GenerateLODMeshes()
    {
        Mesh[] meshes = new Mesh[_lodLevels];
        for(int i = 0; i < meshes.Length; i++)
        {
            Mesh result = new Mesh();
            int vertCount = (_vertexBudget * 2) / (int)Mathf.Pow(2, i);
            int triCount = vertCount * 3 - 6;
            Vector3[] vertices = new Vector3[vertCount];
            int[] tris = new int[triCount];
            Vector2[] uvs = new Vector2[vertCount];

            for (int j = 0; j < vertCount; j+=2)
            {
                vertices[j] = new Vector3(0, j * (i + 1), 0);
                vertices[j+1] = new Vector3(0, j * (i + 1), 1);
                if (j >= vertCount - 2) break;

                int o = j * 3;

                // Triangle 1
                tris[o]     = j;
                tris[o + 1] = j + 1;
                tris[o + 2] = j + 2;

                // Triangle 2
                tris[o + 3] = j + 3;
                tris[o + 4] = j + 2;
                tris[o + 5] = j + 1;
            }

            result.SetVertices(vertices);
            result.SetTriangles(tris, 0);
            meshes[i] = result;
        }
        return meshes;
    }
}


```